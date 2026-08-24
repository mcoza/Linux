# Linux Troubleshooting: Learning and Applied Reasoning

This repository documents how I am developing Linux troubleshooting judgment through hands-on exercises. It is organized around a progression: what I learned, where I applied it, why I selected a command, what its output revealed, and how that evidence guided the next step.

The purpose is not to present a list of commands or claim professional Linux administration experience. It is to show how I use Linux tools to answer operational questions.

## Learning progression

| Stage | Question being answered | Applied work |
|---|---|---|
| [1. File and text investigation](01-file-and-text-investigation.md) | How do I locate, filter, transform, and summarize evidence? | Content searches, log aggregation, column arithmetic, CSV work, and searching virtual filesystems |
| [2. Processes and system state](02-process-and-system-state.md) | Which process or resource explains the symptom, and is the host healthy? | Open-file tracing, process investigation, IPC, CPU/load, memory/OOM, and storage review |
| [3. Services and automation](03-services-and-automation.md) | Did the service or scheduled job run, and did its underlying command succeed? | Service inspection, timers, backups, cleanup jobs, and resource-control troubleshooting |
| [4. Networking and application paths](04-networking-and-application-paths.md) | Which service is listening, where is it exposed, and what dependency comes next? | Service access, port changes and audits, Nginx/backend tracing, TLS, FTP, and database failures |
| [5. Applied investigation](applied-investigations/system-health-review.md) | How do multiple sources of evidence describe one unfamiliar server? | Processes, listeners, CPU/load, memory history, storage, and service relationships |

## Reasoning model

Each command is documented in context:

1. **Situation** — the observable problem or question.
2. **Need** — the information required to continue.
3. **Command choice** — why that tool fits the question.
4. **Interpretation** — what the relevant output means.
5. **Next step** — how the evidence changes the investigation.
6. **Verification** — how the original symptom or expected result is checked.

```text
observe a symptom
      ↓
define what must be known
      ↓
select a command that exposes that state
      ↓
interpret the evidence in context
      ↓
take a targeted next step
      ↓
verify the result
```

## Evidence boundary

The material comes from completed hands-on Linux troubleshooting exercises. Detailed examples are included where the commands, observations, and reasoning were retained accurately. Other completed areas are identified as applied exposure without reconstructing command sequences from memory.

This is personal lab and training work, not professional Linux administration.
