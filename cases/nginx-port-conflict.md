# Nginx Port Conflict and Backend Routing

## Goal

Understand a Linux web-service port conflict without breaking the existing request path.

The useful part of this exercise was connecting the listener, Nginx configuration, loopback backend, and service relationship rather than treating the problem as only "a port is busy."

## What was verified

The Nginx site configuration was inspected under:

```text
/etc/nginx/sites-available/bergen
/etc/nginx/sites-enabled/bergen
```

A directly observed directive was:

```nginx
proxy_pass http://127.0.0.1:8000;
```

That established this request path:

```text
client
  ↓
Nginx
  ↓
proxy_pass
  ↓
127.0.0.1:8000
  ↓
local backend service
```

`127.0.0.1` is loopback, so the backend was intended to be reached from the same host instead of being directly exposed on every interface.

## Why the configuration mattered

The port could not be changed intelligently until the service relationship was understood.

The investigation connected several earlier Linux concepts:

- process and listener discovery
- port ownership/conflicts
- loopback addressing
- configuration-file discovery
- Nginx `sites-available` / `sites-enabled`
- `proxy_pass`
- front-end/backend service relationships

A reusable troubleshooting path is:

```text
port/service symptom
      ↓
inspect listeners and processes
      ↓
locate the active web configuration
      ↓
trace the configured backend
      ↓
compare configuration with the intended service path
```

## Evidence boundary

The SadServers scenario reached the platform's successful completion state. However, the exact final remediation command sequence is not retained accurately enough to reproduce here.

For that reason, this case documents the verified configuration and reasoning above rather than inventing the exact edit, reload command, or final backend value used to complete the scenario.

## What this demonstrates

- Nginx site-configuration inspection
- loopback/backend interpretation
- `proxy_pass` interpretation
- reverse-proxy tracing
- connecting listeners, processes, and configuration into one service path
- recognizing that a port conflict can be a service-relationship problem rather than just a port-number problem
