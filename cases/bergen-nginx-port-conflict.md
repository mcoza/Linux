# Bergen: Nginx Port Conflict and Backend Routing

## Goal

Troubleshoot a Linux web-service port conflict without breaking the existing application path.

This case is useful because the problem could not be treated as only "a port is busy." The investigation had to connect the listening service, the Nginx configuration, the loopback backend, and the request path before changing anything.

## Starting point

The important question became:

```text
What owns the relevant port?
        ↓
How is the existing web service using it?
        ↓
Where is Nginx sending requests?
```

Earlier listener work made it possible to interpret ports and bind addresses rather than treating them as isolated numbers.

## Tracing the active Nginx configuration

The Nginx site was traced through the standard Debian/Ubuntu layout:

```text
/etc/nginx/sites-available/bergen
/etc/nginx/sites-enabled/bergen
```

Searching the configuration showed:

```text
/etc/nginx/sites-enabled/bergen:7:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/bergen:7:       proxy_pass http://127.0.0.1:8000;
```

The important finding was not just `8000`. It was that Nginx was acting as a front end for a backend service bound to loopback.

```text
client request
      ↓
Nginx
      ↓
proxy_pass
      ↓
127.0.0.1:8000
      ↓
backend service
```

## `sites-available` and `sites-enabled`

This exercise clarified a piece of Nginx administration that was newer than most of the surrounding Linux work.

```text
sites-available
→ stores site configuration files

sites-enabled
→ contains the site configurations that Nginx currently loads
```

The matching `bergen` entries were therefore not two unrelated configurations. The enabled path represented the active site configuration relationship.

## Why `127.0.0.1` mattered

The backend target was:

```text
127.0.0.1:8000
```

`127.0.0.1` is loopback, so the backend is intended to be reached from the same machine rather than exposed directly on every interface.

That creates a layered service design:

```text
client-facing listener
        ↓
Nginx reverse proxy
        ↓
local backend listener
```

This changes the troubleshooting question from:

> What process is using this port?

into:

> Which component should own each port, and how does traffic move between those components?

## Troubleshooting sequence

The case brought several earlier Linux skills together:

```text
Port conflict / service problem
        ↓
Inspect listeners and processes
        ↓
Identify the web-service relationship
        ↓
Locate the active Nginx site
        ↓
Inspect proxy_pass
        ↓
Compare Nginx's configured backend with the intended service path
        ↓
Make the targeted configuration change
        ↓
Validate / reload as appropriate
        ↓
Verify the request path works
```

The scenario reached the platform's successful completion state.

The repository intentionally does **not** invent the exact final backend value or reconstruct command output that is no longer retained accurately. The `127.0.0.1:8000` value above is the configuration that was directly observed during the troubleshooting session.

## What this connected

This case combined concepts that had previously appeared separately:

- process and service discovery
- port ownership and conflicts
- bind-address interpretation
- loopback addressing
- configuration-file discovery
- Nginx site activation
- `proxy_pass`
- reverse-proxy/backend relationships
- targeted change followed by verification

## Reusable lesson

A port conflict is often a **service-relationship problem**, not merely a port-number problem.

Before changing a listener or configuration, trace the path:

```text
client
  ↓
front-end listener
  ↓
proxy / routing rule
  ↓
backend listener
  ↓
application
```

That helps avoid "fixing" one port by breaking the service that depends on it.

## Skills demonstrated

- Nginx site-configuration inspection
- `sites-available` / `sites-enabled` interpretation
- reverse-proxy tracing
- `proxy_pass` interpretation
- loopback/backend service understanding
- port-conflict troubleshooting
- connecting listeners, processes, and configuration into one service path
- targeted configuration changes followed by verification
