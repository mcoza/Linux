# Web, Security, and Application Services

## Purpose

Web and application troubleshooting became more useful once service listeners, loopback addresses, and process relationships were already familiar.

The main model is:

```text
client
  ↓
front-end / reverse proxy
  ↓
local backend service
  ↓
application dependency
```

This makes web troubleshooting less about memorizing Nginx directives and more about tracing where a request is supposed to go.

## Nginx site configuration

During the Bergen port-conflict scenario, the active site configuration was traced through:

```text
/etc/nginx/sites-enabled/bergen
/etc/nginx/sites-available/bergen
```

The observed configuration included:

```nginx
proxy_pass http://127.0.0.1:8000;
```

That line connected several earlier Linux concepts:

```text
127.0.0.1
→ loopback/local host

8000
→ backend listening port

proxy_pass
→ Nginx forwards the request to that backend
```

## `sites-available` and `sites-enabled`

On Debian/Ubuntu-style Nginx layouts:

```text
sites-available
→ stores site configuration definitions

sites-enabled
→ represents the site configurations enabled for Nginx
```

A common deployment pattern uses symlinks from `sites-enabled` to files in `sites-available`. The Bergen investigation traced both paths, but this portfolio does not claim that the exact link relationship was independently verified unless it was observed directly.

## Reverse-proxy troubleshooting

The reusable workflow is:

```text
client-facing symptom
        ↓
identify front-end listener/service
        ↓
inspect Nginx site configuration
        ↓
find proxy_pass/backend target
        ↓
compare backend target with actual listener
        ↓
change only the mismatched component
        ↓
reload/verify end-to-end request path
```

The important shift is from:

> "Port 8000 is involved"

into:

> "Nginx is configured to send requests to a specific local backend, so I need to understand both sides of that relationship."

See [Bergen: Nginx Port Conflict](../cases/bergen-nginx-port-conflict.md).

## TLS certificate maintenance

The completed **Geneva — Renew an SSL Certificate** exercise provided hands-on exposure to certificate renewal in a Linux service context.

This supports familiarity with certificate maintenance as an operational task, but the repository does **not** claim deep PKI administration based on one scenario.

Current scope:

```text
certificate renewal exposure
→ practiced

deep certificate-chain design / CA administration / enterprise PKI
→ not claimed here
```

## FTP synchronization troubleshooting

The completed **Edinburgh — FTP catalog sync failure** exercise provided hands-on exposure to diagnosing a failed FTP synchronization workflow.

The exact command sequence is not reconstructed because the retained solution details are incomplete. The portfolio records the completed troubleshooting area without inventing the failure path.

## Database write troubleshooting

The completed **Manhattan — can't write data into database** scenario is the current Medium-difficulty completed exercise.

It supports hands-on exposure to a database/application write failure, but it does not justify claiming production database administration expertise.

A useful layered model for this kind of symptom is:

```text
application tries to write
        ↓
application/process state
        ↓
database service connectivity
        ↓
permissions / ownership / configuration
        ↓
storage / database state
```

The exact Manhattan solution is not reconstructed here because the detailed command trail is not retained.

## Service boundaries matter

A recurring architectural lesson is that an application can be reachable without being directly exposed on every interface.

For example:

```text
external client
    ↓
Nginx / proxy
    ↓
127.0.0.1:<backend-port>
    ↓
local application
```

That is why bind-address interpretation belongs in both networking and web troubleshooting.

## Current level

**Practiced:** Nginx site inspection, `proxy_pass`, loopback/backend interpretation, reverse-proxy tracing.

**Practiced exposure:** TLS certificate renewal, FTP synchronization troubleshooting, database-write troubleshooting.

**Developing:** deeper Nginx administration, TLS/PKI operations, database administration, and production application-service troubleshooting.
