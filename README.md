# Linux Troubleshooting Practice

This repository documents my Linux troubleshooting practice as it developed through hands-on labs. It is intentionally written as a learning progression rather than a command reference or a set of polished production case studies.

The work comes from personal training environments. The goal is to show what I was trying to understand, what evidence I gathered, how I chose the next command, and what I learned when my first assumption was incomplete or wrong.

## Foundation work

Before the troubleshooting exercises became more interconnected, I practiced basic command-line navigation, shell pipelines, SSH-based work, and Debian/Ubuntu package management. That included the normal APT workflow with `apt update`, `apt upgrade`, `apt install`, `apt remove`, `apt search`, and `apt show`, plus the practical distinction between the user-facing `apt` command and the older/script-oriented `apt-get` interface.

Those basics are not presented as a separate project because they became tools used throughout the later troubleshooting work.

## How my troubleshooting approach has developed

### 1. Start with files, text, and command output

My early exercises were mostly about extracting the right piece of information from a large amount of text. I practiced `find`, `grep`, `awk`, `sort`, `uniq`, pipes, arithmetic, and searches under `/proc`.

The first shift in my thinking was from memorizing commands to asking what kind of question I needed to answer:

```text
Where is the object?       → find
What text does it contain? → grep
Which field matters?       → awk / cut
How often does it occur?   → sort / uniq -c
```

[File and text investigation](01-file-and-text-investigation.md)

### 2. Connect a symptom to a process and then to system state

The next group of exercises introduced process ownership and host health. I worked from changing files and open resources into PIDs, process trees, CPU/load, memory, OOM history, storage, and virtual filesystems.

This added a second troubleshooting pattern:

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

### 3. Add identity and permissions to the investigation

User and permission exercises added another question that can sit underneath file, script, or service failures: **which identity is trying to perform the operation, and what access does that identity actually have?**

I practiced checking users, groups, ownership, and permission bits, and learned not to treat `permission denied` as a generic application failure.

[Users and permissions](05-users-and-permissions.md)

### 4. Follow services and automation past the launcher

Later exercises added systemd, cron, timers, service-to-script relationships, and cgroup resource controls.

The important lesson was that these are different claims:

```text
"systemd started the unit successfully"
"the script exited successfully"
"the intended result actually happened"
```

I learned to inspect unit state, the command systemd was configured to launch, logs, scripts, scheduler configuration, paths, lock files, and the final output instead of stopping at a green service state.

[Services and automation](03-services-and-automation.md)

### 5. Trace network and application paths

The networking work started with listeners and bind addresses, then became more useful once I began following ports into processes, services, configuration, and downstream dependencies.

Recent examples included:

- tracing a port conflict from `8000/tcp` to a Django process and its systemd unit
- discovering that Nginx on port 80 proxied requests to that Django backend
- moving the backend without breaking the required web path
- using a `502 Bad Gateway` response as evidence that Nginx was reachable but its upstream was not
- debugging FTP control versus data connections and passive mode
- tracing a local `curl` failure to an iptables OUTPUT rule

[Networking and application paths](04-networking-and-application-paths.md)

## Applied whole-system review

The [whole-system health review](applied-investigations/system-health-review.md) is where several of these areas came together on one unfamiliar server. I used processes, listeners, CPU/load, memory history, and storage to separate two questions that I initially tended to mix together:

1. What is this server doing?
2. Is what I am seeing actually abnormal?

That distinction matters because an unfamiliar process, an open port, or a high utilization value is not automatically a fault.

## Current troubleshooting model

The model I am trying to make habitual is:

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

I am still building speed and familiarity with Linux subsystems. Some later topics—such as Nginx, FTP behavior, systemd timers, and cgroup v2 controls—were first exposures during these labs. They are documented as learning and troubleshooting experience rather than production administration expertise.
