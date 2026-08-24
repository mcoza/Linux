# 4. Networking and Application Paths

This stage developed from identifying individual ports to tracing the relationships among listeners, services, configuration, and application dependencies.

## Reading listeners in context

```bash
ss -tlpn
netstat -tlpn
```

**When I use them:** A service is unreachable, a port appears occupied, or I need to understand which services are exposed.

**What I need from the output:** protocol and socket state, local address and port, bind scope, and owning process when available.

```text
127.0.0.1 → reachable only from the local host
0.0.0.0   → bound on all IPv4 interfaces
:::       → IPv6 unspecified / broadly bound on IPv6
```

TCP `LISTEN` and UDP `UNCONN` describe different protocol behavior. `UNCONN` is not an error by itself because UDP is connectionless.

## Applied port and access investigations

I completed work involving diagnosing network-service access, changing and verifying a service port, auditing a port when standard network tools were unavailable, and resolving a web-service port conflict.

These exercises reinforced that a port number is not the complete diagnosis. The investigation must connect the listener to a process, the process to its configuration, and the configuration to the intended client or dependency.

## Tracing an Nginx backend

**Situation:** A web-service port conflict could not be corrected safely without understanding the existing request path.

The active site configuration was inspected through the Debian/Ubuntu `sites-available` and `sites-enabled` layout. A directly observed directive was:

```nginx
proxy_pass http://127.0.0.1:8000;
```

**Interpretation:** Nginx was forwarding requests to a backend reachable through loopback on the same host. Changing a port without updating the related configuration could break the service chain.

```text
client → Nginx listener → proxy_pass target → local backend
```

**Next step:** Compare the configured target with the actual backend listener, make the smallest consistent correction, validate the configuration, reload the affected service, and verify the request path.

The precise final remediation sequence was not retained, so it is not reconstructed.

## Application and remote-service dependencies

Additional completed work involved TLS certificate renewal, an FTP synchronization failure, and a database write failure.

| Visible problem | Areas that may require evidence |
|---|---|
| Certificate failure | validity period, certificate path, service configuration, reload state |
| Synchronization failure | reachability, authentication, paths, permissions, remote service state |
| Database write failure | application error, database availability, permissions, storage, configuration |

These areas are included as introductory troubleshooting exposure, not claims of deep PKI, FTP, or database administration.

## What changed in my troubleshooting approach

I learned to treat network and application failures as service paths. The goal is not merely to find a port or restart a daemon, but to identify where the actual path diverges from the intended one.
