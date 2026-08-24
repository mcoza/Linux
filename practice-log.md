# Linux Practice Log

This file tracks **confirmed completed** hands-on Linux scenarios. Only exercises that reached SadServers' successful **Well done!** state are counted as solved.

The purpose of the log is to show continued practice while keeping the main portfolio organized around reusable Linux skills rather than individual challenge answers.

## Completed SadServers scenarios

| # | Scenario | Difficulty | Primary area |
|---:|---|---|---|
| 1 | [Saint John — what is writing to this log file?](https://sadservers.com/scenario/saint-john) | Easy | Processes / open files |
| 2 | [Saskatoon — counting IPs](https://sadservers.com/scenario/saskatoon) | Easy | Log analysis / aggregation |
| 3 | [The Command Line Murders](https://sadservers.com/scenario/command-line-murders) | Easy | Search / filtering / command-line investigation |
| 4 | [Taipei — Come a-knocking](https://sadservers.com/scenario/taipei) | Easy | Networking / service access |
| 5 | [Lhasa — Easy Math](https://sadservers.com/scenario/lhasa) | Easy | Text processing / arithmetic |
| 6 | [Minneapolis — Break a CSV file](https://sadservers.com/scenario/minneapolis) | Easy | CSV / text transformation |
| 7 | [Saint Paul — Merge Many CSV files](https://sadservers.com/scenario/st-paul) | Easy | CSV / data manipulation |
| 8 | [Bata — Find in `/proc`](https://sadservers.com/scenario/bata) | Easy | `/proc` / filesystem / content search |
| 9 | [Geneva — Renew an SSL Certificate](https://sadservers.com/scenario/geneva) | Easy | TLS / certificate maintenance |
| 10 | [Linux Server Review — Guided Learning](https://sadservers.com/scenario/linux-server-review) | Easy | Whole-system investigation |
| 11 | [Tokamachi — Troubleshooting a Named Pipe](https://sadservers.com/scenario/tokamachi) | Easy | IPC / named pipes |
| 12 | [Kampot — A New Port](https://sadservers.com/scenario/kampot) | Easy | Ports / services |
| 13 | [Cairo — Time for a Timer](https://sadservers.com/scenario/cairo) | Easy | systemd timers / automation |
| 14 | [Alexandria — The Vanishing Backups](https://sadservers.com/scenario/alexandria) | Easy | Scheduling / backup troubleshooting |
| 15 | [Valladolid — Cleaner not cleaning](https://sadservers.com/scenario/valladolid) | Easy | Automated maintenance troubleshooting |
| 16 | [Porto — Port audit without net tools](https://sadservers.com/scenario/porto) | Easy | Network investigation without standard tools |
| 17 | [Edinburgh — FTP catalog sync failure](https://sadservers.com/scenario/edinburgh) | Easy | FTP / synchronization troubleshooting |
| 18 | [Genova — cgroups problem](https://sadservers.com/scenario/genova) | Easy | cgroups / resource control |
| 19 | [Bergen — Port already in use](https://sadservers.com/scenario/bergen) | Easy | Nginx / port conflict / reverse proxy |
| 20 | [Manhattan — can't write data into database](https://sadservers.com/scenario/manhattan) | Medium | Database / application troubleshooting |

## Totals

- **20 confirmed completed scenarios**
- **19 Easy**
- **1 Medium**

## Detailed portfolio coverage

Not every completed challenge has a reconstructed solution in this repository. That is intentional.

The strongest retained troubleshooting detail currently exists for:

- **Saint John** — process/file relationship and targeted termination
- **Saskatoon** — access-log field extraction and aggregation
- **Lhasa** — column summation, line counting, division, and decimal precision
- **Bata** — `/proc`, `find`, `grep`, `-type f`, `-exec`, and start-of-line matching
- **Linux Server Review** — CPU/load, memory/OOM, disk, process, service, and listener investigation
- **Bergen** — Nginx site configuration, loopback backend routing, and `proxy_pass`

Those details are folded into the reusable notes in `docs/` and the larger investigations in `cases/`.

For the remaining completed scenarios, the completion itself is recorded here, but the exact command sequence is not recreated from memory. This prevents the portfolio from claiming steps that are no longer available with enough confidence to document accurately.

## Not counted as completed

Exercises that were opened, attempted, or discussed but did **not** show the successful completion state are intentionally excluded from the solved count.

This keeps the public record tied to demonstrated completion rather than exposure alone.
