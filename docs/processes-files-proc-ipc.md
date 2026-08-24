# Processes, Files, `/proc`, and IPC

## Purpose

A recurring Linux troubleshooting pattern is that a visible symptom belongs to one resource while the actual cause belongs to a process.

Examples:

```text
log file keeps growing
→ which process is writing to it?

port is occupied
→ which process owns the socket?

file is still in use
→ which process has it open?
```

The useful skill is moving between **resource → process → process state → targeted action**.

## Watching a changing file

```bash
tail -f /var/log/example.log
```

`tail -f` follows new data appended to a file. It is useful when the question is whether a process is still actively writing rather than merely whether the file contains old data.

## Relating files to processes

Tools practiced include:

```bash
fuser
lsof
```

They answer related questions from different directions:

```text
fuser
→ which PIDs are using this file/socket/resource?

lsof
→ what files/sockets does a process have open, or which process has a resource open?
```

This supports a workflow like:

```text
resource shows unwanted behavior
        ↓
identify PID using resource
        ↓
inspect process
        ↓
terminate/change only the responsible process
        ↓
verify resource behavior changed
```

That pattern was practiced in the Saint John scenario, where the goal was to stop the process writing to a log while preserving the log file itself.

## Process discovery

Commands practiced:

```bash
ps
ps auxf
pgrep
```

`ps auxf` is particularly useful when process relationships matter because the tree-style output helps expose parent/child structure rather than showing only a flat PID list.

## Targeted process termination

```bash
kill <PID>
```

The troubleshooting lesson is to identify the correct process first. Deleting a file, restarting unrelated services, or killing broad groups of processes can hide the symptom without addressing the component responsible for it.

## Searching filesystem objects with `find`

`find` selects filesystem objects based on properties such as path, name, and type.

Example concept:

```bash
find /some/path -type f
```

Here:

```text
-type f
→ regular files only
```

`find` and `grep` solve different problems:

```text
find
→ locate/select filesystem objects

grep
→ search text/content
```

They can be combined when the question is "which file under this tree contains this content?"

## `-exec`

A pattern practiced during `/proc` investigation was using `find` to pass matching files to another command.

Conceptually:

```bash
find /path -type f -exec grep -l 'pattern' {} +
```

Important pieces:

```text
-exec
→ run another command on found objects

{}
→ placeholder for found paths

+
→ pass multiple paths per command invocation when possible
```

The exact syntax matters, but so does the reason for it: `find` identifies the candidate files; `grep` examines their contents.

## `/proc`

`/proc` looks like a directory tree, but it exposes runtime kernel/process information through a virtual filesystem.

That explains why it behaves differently from normal disk-backed directories:

- files can appear/disappear as processes and kernel state change
- some entries are permission restricted
- metadata and contents can be generated dynamically

During a search under `/proc/sys`, permission-denied output was part of the investigation rather than proof that the whole search method was wrong.

## Regex anchors

The pattern:

```regex
^secret:
```

was used to distinguish "contains `secret:` somewhere" from "starts with `secret:`".

```text
^
→ start of line
```

That distinction matters when a task has an exact content requirement.

## Named pipes / FIFOs

The Tokamachi scenario provided hands-on exposure to a named-pipe troubleshooting problem.

A FIFO is not a normal persistent data file. It is an IPC mechanism that lets one process write a stream that another process reads.

At the current stage, the portfolio claims **practiced exposure to named-pipe troubleshooting**, not deep IPC expertise. The exact command path from that scenario is not reconstructed here because it is not retained with enough detail.

## Reusable troubleshooting pattern

```text
file / socket / special-file symptom
          ↓
identify object type
          ↓
identify process relationship
          ↓
inspect process state
          ↓
make targeted change
          ↓
verify behavior
```

## Current level

**Practiced** for process/file relationships, process discovery, `/proc` search, and targeted termination.

**Practiced exposure** for named pipes/FIFOs; deeper IPC behavior remains developing.
