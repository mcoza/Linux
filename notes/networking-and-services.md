# Networking and Application Services

This note collects the networking, port, and application-service concepts practiced across the Linux exercises.

## Listeners, ports, and bind addresses

Commands practiced:

```bash
ss -tlpn
netstat -tlpn
```

A listener should be read as more than a port number. The useful questions are:

```text
Which process is listening?
On which address?
Which protocol/state is shown?
Who should be able to reach it?
```

During the Linux Server Review, observed listeners included:

```text
sshd        0.0.0.0:22
HAProxy     0.0.0.0:8000
gotty       :::8080
python3     127.0.0.1:9000
PostgreSQL  127.0.0.1:5432
```

Common bind-address meanings:

```text
127.0.0.1 -> loopback / local host only
0.0.0.0   -> all IPv4 interfaces
:::       -> IPv6 unspecified / all IPv6 interfaces
```

The review also covered TCP `LISTEN` versus UDP `UNCONN`; `UNCONN` is not an error by itself because UDP is connectionless.

## Service relationships

The listener layout supported a broader architecture model:

```text
client-facing service
        ↓
proxy / front end
        ↓
local application
        ↓
local dependency such as a database
```

That made port troubleshooting less about finding a number and more about tracing the service path.

Completed port/networking practice included Taipei, Kampot, Porto, Bergen, and the Linux Server Review. The exact command trails for every short scenario are not retained, so only verified details are documented here.

## Nginx and reverse-proxy tracing

During Bergen, the Nginx site configuration was inspected under the Debian/Ubuntu-style `sites-available` / `sites-enabled` layout. A directly observed directive was:

```nginx
proxy_pass http://127.0.0.1:8000;
```

That showed Nginx forwarding requests to a local backend on loopback.

The reusable investigation path is:

```text
port/service symptom
      ↓
identify listener/process
      ↓
inspect Nginx site configuration
      ↓
find proxy_pass target
      ↓
compare configured backend with the actual service path
```

The scenario was completed, but the exact final remediation command sequence is not retained and is not reconstructed in this portfolio.

See [Nginx Port Conflict](../cases/nginx-port-conflict.md).

## Other service troubleshooting exposure

Completed exercises also provided hands-on exposure to:

- TLS certificate renewal (Geneva)
- FTP synchronization failure (Edinburgh)
- database write failure (Manhattan)

Those exercises support familiarity with the troubleshooting areas, not claims of deep PKI, FTP-server, or database administration expertise.

## What this demonstrates

- listener and port interpretation
- process-to-port mapping
- loopback vs broadly bound services
- basic TCP/UDP state interpretation
- service-path inference
- Nginx site inspection
- `proxy_pass` interpretation
- reverse-proxy/backend tracing
- introductory TLS, FTP, and database-service troubleshooting exposure