# Networking and Service Discovery

## Purpose

Networking became one of the most important developing areas in the Linux work because open ports, bind addresses, and service relationships showed up repeatedly across server review and troubleshooting scenarios.

The core question is not just:

> What ports are open?

It is:

> **Which process is listening, on which address, for which service, and who can reach it?**

## Listener inspection

Commands practiced:

```bash
ss -tlpn
netstat -tlpn
```

A listener line combines several pieces of information:

```text
protocol
state
local address
port
process/PID information
```

During the Linux Server Review, relevant listeners included:

```text
sshd        0.0.0.0:22
HAProxy     0.0.0.0:8000
gotty       :::8080
python3     127.0.0.1:9000
PostgreSQL  127.0.0.1:5432
```

That output was useful because it revealed more than service names: it showed which services were broadly reachable and which were intended to stay local to the host.

## Bind addresses

### Loopback

```text
127.0.0.1
```

Meaning:

```text
local host only
```

A service bound to `127.0.0.1` is not directly listening on the host's other IPv4 interfaces.

### All IPv4 interfaces

```text
0.0.0.0
```

Meaning:

```text
all IPv4 interfaces
```

Whether outside clients can actually connect still depends on routing, firewalling, and the surrounding network, but the process itself is not restricted to loopback.

### IPv6 unspecified address

```text
:::
```

In listener output, this represents the IPv6 unspecified address / all IPv6 interfaces.

## TCP and UDP state

The server review included the distinction between:

```text
TCP LISTEN
```

and UDP output such as:

```text
UNCONN
```

TCP is connection-oriented and has a listening state for servers waiting to accept connections.

UDP is connectionless, so `UNCONN` should not be read as "broken." It describes the socket model rather than a failed connection.

## Service architecture inference

The listener set exposed a useful architecture pattern:

```text
client-facing listener
        ↓
proxy/front-end service
        ↓
loopback application
        ↓
loopback database
```

For example, a local-only application or database can still serve external users indirectly when a front-end service accepts traffic and forwards requests internally.

That model later became directly useful during Nginx reverse-proxy troubleshooting.

## Port → process → configuration

A reusable investigation path is:

```text
port symptom
   ↓
inspect listener
   ↓
identify process/service
   ↓
inspect bind address
   ↓
inspect service configuration
   ↓
trace dependencies/backend
   ↓
verify end-to-end behavior
```

This is stronger than treating `ss` or `netstat` as a final answer.

## Port troubleshooting practice

Completed scenarios included:

- **Taipei — Come a-knocking**
- **Kampot — A New Port**
- **Porto — Port audit without net tools**
- **Bergen — Port already in use**
- **Linux Server Review — Guided Learning**

The exact command trails for Taipei, Kampot, and Porto are not reconstructed here because the retained solution detail is incomplete. Their completion is recorded in the [practice log](../practice-log.md).

Porto is useful conceptually because it required port investigation without the usual network utilities. That reinforces a broader troubleshooting principle:

> Tools are conveniences; the underlying system state still exists when the preferred tool is unavailable.

The portfolio does not invent the specific fallback technique used in that exercise.

## Process association

Tools such as:

```bash
fuser
lsof
```

can complement socket inspection when the question becomes "which process owns this port/resource?"

That links networking back to process investigation.

## Current level

**Practiced:** listener interpretation, common bind addresses, basic TCP/UDP state interpretation, port-to-service mapping, port-conflict troubleshooting.

**Developing:** deeper network diagnosis, routing, packet-level troubleshooting, firewall analysis, DNS troubleshooting, and repeated work without standard network utilities.
