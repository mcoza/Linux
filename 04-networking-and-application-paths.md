# 4. Networking and Application Paths

This stage developed from identifying individual ports to tracing the path behind them: listener → process → service/configuration → dependency.

## Reading listeners in context

```bash
ss -tlpn
netstat -tlpn
```

Useful interpretations:

```text
127.0.0.1 → loopback/local host only
0.0.0.0   → all IPv4 interfaces
:::       → IPv6 unspecified address
```

I learned to look for protocol/state, local address and port, bind scope, and owning process. TCP `LISTEN` and UDP `UNCONN` describe different protocol behavior; `UNCONN` is normal for UDP.

A useful correction to my own reasoning was that an unfamiliar listener is not automatically suspicious. It only becomes relevant when the evidence connects it to the symptom or expected service.

## Port conflict: port → PID → service

A standalone application failed with:

```text
listen tcp4 0.0.0.0:8000: bind: address already in use
```

That error gave me a concrete starting point: something already owned `8000/tcp`.

```bash
fuser -v 8000/tcp
ps -fp <PID>
```

The process was Python running Django on port 8000. Following the PPID and then systemd showed that `django.service` launched it with an `ExecStart` argument containing `0.0.0.0:8000`.

```text
port 8000
→ owning PID
→ Python/Django process
→ parent/systemd
→ django.service configuration
```

That explained why killing the PID was not yet a justified fix: the process was managed and might also be required by another component.

## First useful Nginx troubleshooting path

The same exercise required the web application on port 80 to keep working. `curl localhost:80` returned the expected page, so I first established that as a working baseline.

I had not worked much with Nginx before this exercise. The useful commands were:

```bash
sudo nginx -T
sudo nginx -T | grep -nE 'listen|proxy_pass|8000'
sudo grep -Rni 'proxy_pass.*8000' /etc/nginx
```

`nginx -T` showed the configuration Nginx was actually loading. The filtered output exposed:

```nginx
listen 80 default_server;
proxy_pass http://127.0.0.1:8000;
```

That made the path visible:

```text
client → Nginx :80 → Django :8000
```

`proxy_pass` meant Nginx was accepting the request on port 80 and forwarding it to the local Django backend on port 8000.

Because the standalone program had to keep port 8000, the safe change was:

```text
client → Nginx :80 → Django :8001
standalone           → :8000
```

Both configurations had to change: the systemd unit that launched Django and the Nginx `proxy_pass` target.

The site configuration appeared in both `sites-available` and `sites-enabled`. Checking the enabled entry with `ls -l` showed the symlink relationship before I edited the underlying configuration.

After changing the Django unit, systemd had to reread the unit and restart Django. After changing Nginx, I validated and reloaded its configuration:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

That distinction was another version of a pattern I had already seen with systemd: changing a file on disk does not mean the running daemon is already using the new configuration.

## What the 502 taught me

During the change, `curl localhost:80` returned `502 Bad Gateway` while Nginx still pointed at the old backend port.

That was useful evidence:

```text
client reached Nginx :80      ✓
Nginx reached its backend     ✗
```

So a 502 from Nginx pushed the next investigation toward the configured upstream/backend rather than back toward whether port 80 was listening.

## FTP: login worked, transfer failed

Another exercise used an FTP synchronization script. The client connected and authenticated successfully but failed when the file transfer began because the script forced active mode with:

```text
passive off
```

The server responded that `PORT/EPRT` was not allowed and passive mode should be used.

This introduced the fact that FTP separates control and data traffic:

```text
control connection → login and FTP commands, normally TCP 21
data connection    → separate connection for the actual transfer
```

Successful login therefore did not prove that the data-transfer path was working. Removing the forced active mode allowed the transfer to complete, after which the dependent API service could be retried.

## Localhost still uses the network stack

A separate health-check script used:

```bash
curl http://localhost
```

and hung because an iptables OUTPUT rule blocked traffic to `127.0.0.1:80`.

The useful inspection path included the OUTPUT chain rather than assuming localhost traffic bypassed firewall behavior:

```bash
sudo iptables -L OUTPUT -n --line-numbers
```

That reinforced that loopback traffic stays on the same machine but still goes through host networking behavior that can be filtered.

## Additional exposure

Other completed labs included changing and auditing service ports, TLS/certificate troubleshooting, and database-related failures. I treat those as troubleshooting exposure rather than deep Nginx, FTP, PKI, or database administration experience.

## What changed in my troubleshooting approach

I started this area mostly thinking in terms of open ports. The more useful model became:

```text
symptom
→ endpoint/socket
→ owning process
→ service or configuration
→ next application dependency
→ smallest justified change
→ verify the original behavior
```

The Nginx and FTP details were new, but they became easier to understand once they were connected to the process, systemd, port, and `curl` concepts I had already practiced.
