# Linux System Troubleshooting

An ongoing hands-on Linux troubleshooting portfolio built from completed lab work and guided system investigations.

This repository is intentionally **cumulative** rather than a collection of one-off challenge solutions. Small exercises feed into broader troubleshooting skills, while larger multi-component investigations become case studies.

## Current coverage

### Command line and data processing

- searching and filtering text
- counting and aggregating log data
- shell-based arithmetic and field processing
- CSV manipulation and merging
- filesystem and `/proc` investigation

### Processes, files, and IPC

- identifying processes interacting with files
- process inspection and termination
- file/process relationships through tools such as `ps`, `fuser`, and `lsof`
- named-pipe troubleshooting

### System analysis

- CPU and load interpretation
- memory review and historical OOM investigation
- disk-capacity checks
- process/service discovery
- cgroup-related troubleshooting

### Networking and services

- TCP/UDP listener inspection
- interpreting `127.0.0.1`, `0.0.0.0`, and `:::` bind addresses
- mapping ports to services and processes
- port auditing with and without standard networking tools
- troubleshooting port conflicts
- tracing reverse-proxy traffic to backend services

### Services and automation

- scheduled task troubleshooting
- systemd timer investigation
- backup/automation failures
- service configuration and verification

### Security and application services

- SSL certificate renewal
- Nginx configuration and reverse proxying
- FTP synchronization troubleshooting
- database write troubleshooting

## Troubleshooting approach

The goal is not to memorize commands. The recurring workflow is:

```text
Symptom
  ↓
Observe current state
  ↓
Identify the responsible component
  ↓
Inspect process / service / configuration / logs
  ↓
Form a hypothesis
  ↓
Make a targeted change
  ↓
Verify the result
```

A second pattern that became important during the server-review work is separating two questions:

```text
What is this system doing?

from

Is this system healthy?
```

That distinction helps avoid jumping directly from a high CPU, memory, or network value to a conclusion without understanding the role of the system first.

## Case studies

- [Linux Server Review](cases/linux-server-review.md) — system purpose, CPU/load, memory/OOM, disk, processes, and listening services
- [Bergen: Nginx Port Conflict](cases/bergen-nginx-port-conflict.md) — port conflict, service discovery, Nginx configuration, `proxy_pass`, and backend routing

## Practice log

A compact record of completed hands-on scenarios is kept in [practice-log.md](practice-log.md).

The practice log is evidence of repetition and coverage; the case studies are where larger troubleshooting chains are documented in detail.

## How this repository grows

When new work is completed:

1. Add the completed exercise to the practice log.
2. Update the relevant skill area in this README when the exercise adds reusable knowledge.
3. Create a case study only when the work connects multiple components or demonstrates a substantial troubleshooting process.

This keeps the repository maintainable as Linux skills expand without turning every short exercise into a separate project page.

## Scope

This is personal lab and training work. It is not production or professional work experience.
