# 4. Networking and Application Paths

This stage developed from identifying individual ports to tracing the relationships among listeners, processes, services, configuration, and application dependencies.

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

One important correction to my own troubleshooting approach was learning that an unfamiliar listener is not automatically suspicious. A port becomes meaningful only when it is connected to the reported symptom, expected service, configuration, or application behavior.

## Port conflict: follow the port instead of guessing

A standalone application failed with:

```text
listen tcp4 0.0.0.0:8000: bind: address already in use
```

That error provided a concrete fact: the application needed TCP port 8000 and the kernel would not let it bind because something else already owned it.

Instead of guessing from a list of open ports, I could ask directly who was using that resource:

```bash
fuser -v 8000/tcp
ps -fp <PID>
```

The process was Python running Django with:

```text
python3 manage.py runserver 0.0.0.0:8000
```

Following the parent process and then systemd showed that `django.service` was responsible for launching the backend on port 8000.

The path at that point was:

```text
port 8000
   ↓
fuser / ss
   ↓
PID
   ↓
ps / PPID
   ↓
django.service
   ↓
ExecStart ... :8000
```

That was more useful than simply killing the Python PID, because the service manager could recreate it and the process might be required by another component.

## Learning how Nginx fit into the path

The same exercise required keeping an existing web application on port 80 working. `curl localhost:80` returned the expected page, so port 80 became a requirement to preserve rather than something to change blindly.

I inspected Nginx with:

```bash
sudo nginx -T
```

and narrowed the output to the relevant directives. The active configuration showed:

```nginx
listen 80 default_server;
proxy_pass http://127.0.0.1:8000;
```

This was my first useful interaction with Nginx as part of a troubleshooting chain. The directive made the application path concrete:

```text
client
  ↓
Nginx :80
  ↓
proxy_pass
  ↓
Django :8000
```

That explained why simply killing Django would have freed port 8000 but broken the required website.

The safe change was to move Django to a different free port and update Nginx to point to the same new backend:

```text
client → Nginx :80 → Django :8001
standalone           → :8000
```

The systemd unit and Nginx config had to agree on the new backend location.

## Configuration on disk versus running state

This exercise also tied together a pattern I had already seen with systemd.

Changing the Django unit file did not automatically move the existing Python process. The unit had to be reloaded and the service restarted.

Changing the Nginx configuration also did not automatically change the running Nginx worker behavior. Before applying it, I checked the syntax:

```bash
sudo nginx -t
```

and then reloaded Nginx:

```bash
sudo systemctl reload nginx
```

The general pattern became:

```text
edit configuration
      ↓
validate/reload the manager or daemon as required
      ↓
verify the actual runtime state
```

## Using a 502 as evidence

During the port move, `curl localhost:80` returned:

```text
502 Bad Gateway
```

That did not mean Nginx itself was unreachable. In this context, the response proved that the request reached Nginx, but Nginx could not successfully reach the configured backend.

The temporary mismatch was:

```text
Nginx :80 → old backend :8000   X
Django    → new backend :8001
```

Updating `proxy_pass` and reloading Nginx restored the request path.

This gave me a new symptom association:

```text
Nginx 502
→ front end is responding
→ inspect the configured upstream/backend next
```

## Nginx configuration layout

The exercise also introduced the Debian/Ubuntu layout:

```text
/etc/nginx/sites-available/
/etc/nginx/sites-enabled/
```

I used `grep` to locate the active `proxy_pass` directive and `ls -l` to confirm that the enabled site was a symlink to the available configuration file before editing the source configuration.

That was a new subsystem-specific detail, not something I could have inferred just from knowing ports or processes.

## FTP synchronization: control versus data connection

Another exercise involved a script that successfully connected and authenticated to an FTP server but failed when it tried to transfer a file.

The script explicitly contained:

```text
passive off
```

and the server returned an error indicating that active-mode `PORT/EPRT` was not allowed and passive mode should be used.

The important concept was that FTP uses separate connections:

```text
control connection → usually TCP 21, login and commands
data connection    → separate connection used for file transfer
```

Because login succeeded, the control channel was working. The failure appeared when the data connection was required. Removing the forced active mode allowed the client to use passive mode and complete the transfer.

The synchronization service then succeeded, and the API that depended on the synchronized catalog had to be retried.

That exercise connected:

```text
systemd service
→ Bash script
→ FTP control/data behavior
→ local file
→ dependent API
```

## Localhost is still part of the network stack

A separate health-check exercise used:

```bash
curl http://localhost
```

and hung because an iptables OUTPUT rule blocked traffic to `127.0.0.1:80`.

That reinforced that loopback traffic is local but still travels through the host networking stack and can be affected by firewall policy.

## Additional exposure

Other completed labs included changing and auditing service ports, TLS/certificate troubleshooting, and database-related failures. Those are included as troubleshooting exposure rather than claims of deep Nginx, FTP, PKI, or database administration.

## What changed in my troubleshooting approach

Earlier, I tended to view networking as "what port is open?" The later exercises made the path more useful:

```text
reported network/application symptom
        ↓
which socket or endpoint is involved?
        ↓
which process owns it?
        ↓
which service or configuration controls that process?
        ↓
what client, proxy, file, or backend depends on it?
        ↓
change the smallest part of the path
        ↓
verify the original application behavior
```

The Nginx work was genuinely new, but it became understandable once it was connected to familiar process, port, systemd, and `curl` evidence.
