# Bergen: Nginx Port Conflict

## Goal

Troubleshoot a Linux web-service port conflict without breaking the existing application path.

This case became more useful than a simple "port already in use" exercise because it required tracing how Nginx was connected to the backend application.

## Investigation

The active Nginx site configuration was traced through:

```text
/etc/nginx/sites-enabled/bergen
/etc/nginx/sites-available/bergen
```

Searching the configuration showed:

```text
/etc/nginx/sites-enabled/bergen:7:        proxy_pass http://127.0.0.1:8000;
/etc/nginx/sites-available/bergen:7:      proxy_pass http://127.0.0.1:8000;
```

The important discovery was not just the port number. It was understanding the request path:

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

## Nginx configuration relationship

This exercise also clarified the purpose of the two common Debian/Ubuntu Nginx directories:

```text
sites-available
→ stores site configuration files

sites-enabled
→ contains the configurations currently enabled by Nginx
```

Seeing the same `proxy_pass` entry in both locations helped connect the enabled site to its underlying configuration rather than treating the files as unrelated duplicates.

## Port and bind-address context

Earlier listener work made the `127.0.0.1` address meaningful:

```text
127.0.0.1:8000
```

means the backend is bound to loopback and is intended to be reached locally, while Nginx provides the client-facing layer.

That changes the troubleshooting question from:

> "What owns port 8000?"

into:

> "Which service should own the backend port, and where is Nginx configured to send requests?"

## Resolution workflow

The troubleshooting sequence was:

```text
Port/service problem
      ↓
Inspect listening services and processes
      ↓
Identify Nginx as part of the request path
      ↓
Locate the active site configuration
      ↓
Inspect proxy_pass
      ↓
Compare the configured backend with the service that should receive traffic
      ↓
Make the targeted Nginx configuration change
      ↓
Reload/verify the service path
```

The scenario reached the successful completion state. The exact final backend value is intentionally not reconstructed here from memory; the documented value above is the configuration that was directly observed during troubleshooting.

## Why this case matters

This was one of the first exercises where several earlier Linux concepts converged:

- processes
- ports
- loopback addressing
- service listeners
- configuration files
- Nginx
- reverse proxies
- backend service relationships

The main reusable lesson is to trace **service relationships** instead of treating each port or configuration file as an isolated object.

## Skills demonstrated

- Nginx site-configuration inspection
- `sites-available` / `sites-enabled` relationship
- reverse-proxy tracing
- `proxy_pass` interpretation
- loopback/backend service understanding
- port-conflict troubleshooting
- targeted configuration changes followed by verification
