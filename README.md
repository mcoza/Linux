# Linux Troubleshooting Practice

This repository documents how my Linux troubleshooting has developed through the Linux Upskill Challenge, SadServers, and related hands-on labs. It is intentionally written as a learning progression rather than a command reference or a collection of polished production case studies.

The goal is to show what I was trying to understand, what evidence I gathered, why I chose the next command, and what I learned when my first assumption was incomplete or wrong.

This is personal training work, not professional Linux administration experience.

## Foundation work

Before the troubleshooting exercises became more interconnected, I practiced command-line navigation, SSH-based work, shell pipelines, files and permissions, and Debian/Ubuntu package management.

That included the normal APT workflow with `apt update`, `apt upgrade`, `apt install`, `apt remove`, `apt search`, and `apt show`, plus the practical distinction between the user-facing `apt` command and the older/script-oriented `apt-get` interface.

Those basics are not presented as a separate project because they became tools used throughout the later troubleshooting work.

## How my troubleshooting approach developed

### 1. Start with files, text, and command output

My early exercises were mostly about extracting the right piece of information from a large amount of text. I practiced `find`, `grep`, `awk`, `sort`, `uniq`, pipes, arithmetic, log analysis, and searches under `/proc`.

The first shift was from memorizing commands to asking what kind of information I needed:

```text
Where is the object?       → find
What text does it contain? → grep
Which field matters?       → awk / cut
How often does it occur?   → sort / uniq -c
```

[File and text investigation](01-file-and-text-investigation.md)

### 2. Connect a symptom to a process and then to system state

The next exercises introduced process ownership and host health. I worked from changing files and open resources into PIDs, process trees, CPU/load, memory, OOM history, storage, and virtual filesystems.

This added another troubleshooting path:

```text
known resource or symptom
        ↓
which process owns it?
        ↓
what is that process doing?
        ↓
is the problem local to the process or part of wider system pressure?
```

Tools here included `ps`, `pgrep`, `fuser`, `lsof`, `top`, `htop`, `free`, `vmstat`, `df`, `du`, `lsblk`, `dmesg`, and `journalctl`.

[Processes and system state](02-process-and-system-state.md)

### 3. Follow services and automation past the launcher

Later exercises added systemd, cron, timers, service-to-script relationships, service dependencies, and cgroup resource controls.

The important lesson was that these are different claims:

```text
"systemd started the unit successfully"
"the script exited successfully"
"the intended result actually happened"
```

I learned to inspect unit state, the command systemd was configured to launch, logs, scripts, scheduler configuration, paths, lock files, dependencies, and the final output instead of stopping at a green service state.

[Services and automation](03-services-and-automation.md)

### 4. Trace network and application paths

The networking work started with listeners and bind addresses. It became more useful once I began following ports into processes, services, configuration, and downstream dependencies.

Examples included:

- tracing a port conflict from `8000/tcp` to a Django process and its systemd unit
- discovering that Nginx on port 80 proxied requests to that Django backend
- moving the backend without breaking the required web path
- using a `502 Bad Gateway` response as evidence that Nginx was reachable but its upstream was not
- debugging FTP control versus data connections and passive mode
- tracing a local `curl` failure to an iptables OUTPUT rule

[Networking and application paths](04-networking-and-application-paths.md)

### 5. Keep identity and permissions in the path

User and permission exercises added another question that can sit underneath file, script, and service failures: **which identity is trying to perform the operation, and what access does that identity actually have?**

I practiced checking users, groups, ownership, and permission bits, and learned not to treat `permission denied` as a generic application failure. This became a cross-cutting check rather than an isolated permissions topic.

[Users and permissions](05-users-and-permissions.md)

## Applied whole-system review

The [whole-system health review](applied-investigations/system-health-review.md) is where several of these areas came together on one unfamiliar server. I used processes, listeners, CPU/load, memory history, and storage to separate two questions that I initially tended to mix together:

1. What is this server doing?
2. Is what I am seeing actually abnormal?

That distinction matters because an unfamiliar process, an open port, or a high utilization value is not automatically a fault.

## Current troubleshooting model

The model I am trying to make repeatable is:

```text
reproduce the symptom
        ↓
extract a concrete fact from the error or requirement
        ↓
choose a command that exposes that state
        ↓
interpret the relevant output
        ↓
follow the relationship to the next component
        ↓
make the smallest justified change
        ↓
verify the original requirement
```

The progression in this repository is the part I want to preserve. Early work was mostly command and output interpretation. Later work increasingly required connecting multiple subsystems—processes, systemd, scripts, sockets, firewalls, reverse proxies, schedulers, permissions, and resource controls—without assuming that an unfamiliar component was automatically the problem.

Some later topics, such as Nginx, FTP behavior, systemd timers, and cgroup v2 controls, were first exposures during these labs. They are documented as learning and troubleshooting experience rather than production administration expertise.
