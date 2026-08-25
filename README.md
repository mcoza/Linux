# Linux Troubleshooting Lab Portfolio

This repository documents the Linux troubleshooting skills I have practiced through hands-on labs, including SadServers and the Linux Upskill Challenge.

The focus is not the individual lab names. The focus is what I can recognize, inspect, connect, and verify on a Linux system.

This is personal lab and training work, not professional Linux administration experience.

## What I can work with

| Area | Knowledge and hands-on capability |
|---|---|
| **Files and text** | Locate files, search contents, extract fields, count/group values, process structured text, verify intermediate results, and work with `/proc` as a live virtual filesystem |
| **Processes and resources** | Identify PIDs and PPIDs, trace a file or port to its owning process, inspect process trees, and connect a running process back to the thing that launched it |
| **System state** | Check CPU/load, memory, OOM history, filesystem usage, block devices, and distinguish current state from earlier failure evidence |
| **Services and automation** | Inspect systemd units, `ExecStart`, runtime state, logs, cron jobs, systemd timers, service dependencies, and introductory cgroup v2 controls |
| **Networking and applications** | Inspect listeners and bind addresses, trace port conflicts, follow reverse-proxy paths, interpret a `502`, inspect loopback/firewall behavior, troubleshoot DNS responses, and distinguish FTP control from data connections |
| **Users and permissions** | Inspect identity, groups, ownership, and permission bits and use that information to investigate access failures |
| **Package management** | Refresh APT metadata, inspect/search packages, install/remove software, apply upgrades, and distinguish `apt` from `apt-get` use cases |

## What that lets me do

The individual commands become useful when they are connected to a question.

```text
A program says a port is already in use
→ identify the exact port
→ inspect the listener
→ identify its PID
→ inspect the process and parent
→ follow it to service/configuration if needed
```

```text
A service fails
→ inspect its current systemd state
→ read the command in ExecStart
→ inspect logs
→ follow the launched program/script to the next failure
```

```text
A scheduled task is not producing its result
→ determine whether cron or a systemd timer launches it
→ inspect the configured command/path
→ run the task manually
→ separate scheduler problems from script/application problems
```

```text
A machine reports no space
→ use df to identify the affected filesystem
→ use du to identify visible consumers
→ investigate open/deleted files if filesystem and visible-file usage disagree
```

```text
A hostname or web request fails
→ use the exact response or error as evidence
→ inspect the relevant DNS, listener, firewall, proxy, or backend state
→ verify the original request again after the change
```

## Troubleshooting method

The method I am trying to make repeatable is:

```text
reproduce the symptom
        ↓
extract a concrete fact from the error or requirement
        ↓
ask what mechanism could produce that fact
        ↓
choose a command that exposes that mechanism/state
        ↓
interpret only what the output actually proves
        ↓
follow the relationship to the next component
        ↓
make the smallest justified change
        ↓
verify the original requirement
```

A few rules are especially important:

- **Unknown does not mean bad.** An unfamiliar process or port is only relevant when evidence connects it to the symptom.
- A successful launcher does not prove the task produced the correct result.
- A current healthy-looking snapshot does not prove that an earlier failure did not happen.
- A PID, port, service, or log entry is usually a waypoint to the next relationship, not automatically the root cause.
- Verification means rerunning the original requirement, not merely seeing that an edit or restart succeeded.

## Knowledge areas and examples

### [1. File and text investigation](01-file-and-text-investigation.md)

What I know: `find`, `grep`, `awk`, `cut`, `sort`, `uniq`, pipes, redirection, numeric aggregation, and `/proc` searching.

What I can do with it: turn a vague question such as “which value occurs most often?” into an extract → group → count → rank pipeline, or combine `find` and `grep` when I need both the location of a file and a match inside it.

### [2. Processes and system state](02-process-and-system-state.md)

What I know: PID/PPID relationships, `ps`, `fuser`, `lsof`, `top`/`htop`, CPU/load context, memory state, OOM history, `df`/`du`, and basic FIFO behavior.

What I can do with it: start from a known resource or utilization symptom, identify the responsible process, follow its parent/manager, and determine whether the issue is local to one process or part of wider host pressure.

### [3. Services and automation](03-services-and-automation.md)

What I know: systemd manager/unit/process relationships, `systemctl`, `journalctl`, `ExecStart`, reload/restart behavior, cron, timers, dependencies, and introductory cgroup v2 controls.

What I can do with it: distinguish service-manager state from application behavior, trace a unit into the program or script it launches, diagnose why work runs manually but not automatically, and verify whether configured resource controls are actually applied.

### [4. Networking and application paths](04-networking-and-application-paths.md)

What I know: TCP/UDP listener state, bind addresses, socket ownership, reverse proxying, Nginx upstreams, loopback filtering, DNS troubleshooting, FTP control/data behavior, and basic TLS/TCP inspection.

What I can do with it: turn network errors into a path to investigate—for example, bind error → port → PID → process → service, or `502` → reachable proxy → failed/misconfigured backend path.

### [5. Users and permissions](05-users-and-permissions.md)

What I know: UID/GID/group context, file ownership, permission bits, and numeric permission notation.

What I can do with it: investigate whether an operation fails because of the executing identity or the permissions on the resource it is trying to access.

### [6. Debian/Ubuntu package management](06-package-management.md)

What I know: the difference between refreshing package metadata and changing installed packages, the normal APT install/remove/search/show workflow, and the practical distinction between `apt` and `apt-get`.

What I can do with it: inspect package availability before changing the system, install or remove required tools, and reason about `apt update` and `apt upgrade` as separate steps rather than interchangeable commands.

### [Applied unfamiliar-server review](applied-investigations/system-health-review.md)

This pulls several areas together: process inventory, listeners, tentative application relationships, CPU/load context, memory/OOM history, and storage state. It demonstrates how I can inspect an unfamiliar Linux host without treating every unfamiliar process or open port as a fault.

## Scope

The examples in this repository come from controlled lab environments. They demonstrate hands-on troubleshooting practice and growing Linux system understanding; they are not presented as production administration experience.
