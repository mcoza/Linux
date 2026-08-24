# 1. File and Text Investigation

My earliest Linux troubleshooting exercises were mostly about getting the right fact out of files, logs, and command output. The commands themselves were usually familiar faster than the decision of **which command answered which question**.

That became the first pattern I tried to make repeatable:

```text
Where is it?        → find
What does it say?   → grep
Which field matters?→ awk / cut
How often?          → sort / uniq -c
What is the result? → verify the final value
```

## Finding an object versus searching its contents

One exercise required finding a file under `/proc/sys` whose contents began with a specific value.

```bash
find /path -type f
find /path -type f -exec grep -l '^pattern:' {} +
```

What I learned was the distinction between two questions:

- `find` locates filesystem objects.
- `grep` searches text inside them.

The `^` anchor restricts the match to the start of a line, and `-type f` avoids treating directories as regular files.

Working under `/proc` also introduced a useful complication: it is a virtual filesystem exposing live kernel/process state. Permission errors and changing entries are part of that environment, so they do not automatically mean the search command is wrong.

## Turning a log into a count

Another exercise asked which source address appeared most often in an access log.

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

The pipeline made more sense once I stopped treating it as one long command:

```text
extract field
→ group identical values
→ count them
→ rank the counts
→ inspect the top result
```

In the exercise, the highest-frequency address appeared 482 times. That number answered the counting question; it did not by itself prove the traffic was malicious or abnormal.

The same style of text processing later showed up in HTTP status-code counting and word-frequency exercises.

## Arithmetic from structured text

I also used `awk` and `bc` to calculate values from numeric columns.

```awk
sum += $2
```

This exercise was useful because I initially focused on the final arithmetic. Breaking it into smaller checks was more reliable:

```text
correct field?
→ correct running total?
→ correct record count?
→ correct division/precision?
```

That same habit carried into later troubleshooting: verify intermediate evidence instead of waiting until the end to discover that the first assumption was wrong.

## Other early command-line practice

Across the early exercises I also used:

- `head` and `tail` to narrow output
- `grep -c` to count matches
- `cut` and `awk` for field extraction
- `md5sum` for file verification
- shell pipes and redirection
- CSV filtering, transformation, and merging
- basic package-management work with APT

I completed additional file/search exercises whose exact command trails were not retained well enough to reconstruct. I keep those as exposure rather than inventing a polished walkthrough from memory.

## What changed in my troubleshooting approach

At the beginning, `find`, `grep`, `awk`, `sort`, and `uniq` felt like separate commands to remember. The useful change was learning to start with the information I needed and then choose the tool that exposed it.

That became the base for later work where the "thing being searched" was no longer just text—it could be a process, socket, service, log history, or configuration file.
