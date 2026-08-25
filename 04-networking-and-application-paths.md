# 4. Networking and Application Paths

This section documents the networking and application-path concepts I can inspect and connect on a Linux host.

## What I know

I have hands-on practice with:

```text
ss / netstat   → inspect listening sockets and network state
fuser          → map a known port to a process
curl           → test an HTTP endpoint
iptables       → inspect host firewall/NAT rules
dig            → query DNS
nginx -T       → inspect the Nginx configuration actually loaded
nginx -t       → validate Nginx configuration before reload
OpenSSL tools  → inspect basic TLS/certificate behavior
/dev/tcp       → test TCP connectivity from Bash when normal tools are unavailable
```

I also understand these address/listener basics:

```text
127.0.0.1 → loopback/local host only
0.0.0.0   → all IPv4 interfaces
:::       → IPv6 unspecified address
TCP LISTEN → server socket waiting for TCP connections
UDP UNCONN → normal unconnected UDP socket state
```

An unfamiliar listener is not automatically a problem. I need another fact that connects it to the reported symptom.

## What I can do with those tools

### Trace a bind failure to the process occupying the port

If an application reports:

```text
listen tcp4 0.0.0.0:8000: bind: address already in use
```

I know the failing program attempted to bind TCP port 8000. That gives me a specific resource to inspect.

```bash
sudo ss -ltnp
sudo fuser -v 8000/tcp
ps -fp <PID>
```

The commands answer a sequence of questions:

```text
ss    → is something listening on :8000, and what process owns it?
fuser → which PID is using 8000/tcp?
ps    → what command is that PID running, and what is its PPID?
```

In the lab example, port 8000 belonged to a Python process running a Django development server. Following the PPID showed that the process was managed rather than simply started manually.

That matters because:

```text
port conflict confirmed
≠ kill the PID immediately
```

The next question is what launched that process and whether another application depends on it.

### Follow a reverse-proxy path

A web service on port 80 was working while its backend occupied the port needed by another program.

Nginx configuration inspection showed:

```nginx
listen 80 default_server;
proxy_pass http://127.0.0.1:8000;
```

That makes the path:

```text
client
  ↓ HTTP :80
Nginx
  ↓ proxy_pass
Django :8000
```

Useful inspection commands included:

```bash
sudo nginx -T
sudo nginx -T | grep -nE 'listen|proxy_pass|8000'
sudo grep -Rni 'proxy_pass.*8000' /etc/nginx
```

To free port 8000 without breaking the web application, both ends of the backend relationship had to agree on the new backend port:

```text
client → Nginx :80 → Django :8001
standalone program  → :8000
```

After changing the service launch configuration and Nginx upstream, I used:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

This reinforced that editing a configuration file does not mean the running daemon has already adopted it.

### Use an HTTP response to decide which hop to inspect

A temporary response of:

```text
502 Bad Gateway
```

provided useful evidence.

Because Nginx generated the HTTP response, the request reached the front end. The failing part was farther down the path:

```text
client → Nginx :80  ✓
Nginx → backend     ✗
```

That tells me to inspect the upstream/backend configuration or listener rather than start over by asking whether Nginx itself is reachable.

### Distinguish FTP control from data transfer

I understand that FTP uses separate connections for command/control traffic and file data.

```text
control connection → authentication and FTP commands, normally TCP 21
data connection    → separate connection used for transfer
```

A synchronization script could connect and authenticate successfully but still fail when the file transfer began because it forced active mode:

```text
passive off
```

The server rejected the active-mode `PORT/EPRT` behavior and required passive mode.

The script also demonstrated how Bash can feed commands to an interactive FTP client with a here-document:

```bash
ftp -inv "$HOST" <<EOF
user ${USER} ${PASS}
binary
get ${REMOTE} ${TMP}
bye
EOF
```

Inside the FTP client:

```text
get remote-file local-file
```

so the transfer path and destination can be reasoned about separately from the Bash wrapper.

The reusable lesson is:

```text
FTP login succeeded
≠ FTP file transfer succeeded
```

### Investigate localhost traffic like network traffic

A local health check used:

```bash
curl http://localhost
```

and hung because an `iptables` OUTPUT rule blocked traffic to `127.0.0.1:80`.

I inspected the OUTPUT chain with:

```bash
sudo iptables -L OUTPUT -n --line-numbers
```

That demonstrated that loopback traffic remains on the same host but still travels through the host networking stack and can be filtered.

### Investigate DNS separately from application reachability

If a hostname query returns `SERVFAIL`, I know the DNS server responded but could not successfully answer the query.

```bash
dig <hostname>
```

I also used a DNS server's zone-check command to validate zone data. This introduced an important DNS naming rule:

```text
name with trailing dot    → absolute DNS name
name without trailing dot → may be relative to the current zone
```

That gives me another useful distinction:

```text
DNS server reachable
≠ DNS zone/configuration valid
```

## How errors narrow my first inspection

```text
"address already in use" → inspect the exact port with ss/fuser
"connection refused"     → check whether the expected listener exists
"hang" / "timeout"       → investigate path/filtering/response as possibilities
"502 Bad Gateway"        → front end answered; inspect backend/upstream
`SERVFAIL`                → inspect DNS response and zone/server configuration
FTP login works but transfer fails → separate control from data path
```

These are starting directions, not guaranteed root causes.

## What this lets me do

I can treat a networked application as a path rather than a single service:

```text
client/request
→ local or remote endpoint
→ listener/socket
→ owning process
→ service/configuration
→ proxy/firewall/DNS/dependency
→ next hop
→ verification from the original client request
```
