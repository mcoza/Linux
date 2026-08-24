# Package Management with APT

## Purpose

This page captures the package-management concepts practiced while working through Debian/Ubuntu Linux systems.

The important lesson was not memorizing package commands individually. It was understanding the difference between **refreshing package metadata**, **changing installed software**, and **inspecting package information**.

## Core workflow

```text
repository metadata
      ↓
apt update
      ↓
local package index refreshed
      ↓
search / inspect / install / upgrade / remove
```

## Commands practiced

### Refresh available package metadata

```bash
sudo apt update
```

`apt update` does **not** upgrade installed packages. It refreshes the local information about what package versions are available from configured repositories.

### Upgrade installed packages

```bash
sudo apt upgrade
```

This applies available upgrades to installed packages using the refreshed package metadata.

The distinction is:

```text
apt update
→ update knowledge of available packages

apt upgrade
→ update installed packages
```

### Install software

```bash
sudo apt install <package>
```

### Remove software

```bash
sudo apt remove <package>
```

### Search repositories

```bash
apt search <term>
```

### Inspect package information

```bash
apt show <package>
```

This is useful before installation when the question is not just "does this package exist?" but also what it provides, its version, dependencies, and metadata.

## `apt` vs `apt-get`

One of the early points of confusion was whether `apt-get` had been replaced or deprecated.

The practical distinction is:

```text
apt
→ newer human-oriented command-line interface
→ combines common package-management operations
→ output is designed for interactive use

apt-get
→ older, stable command interface
→ still supported
→ commonly preferred in scripts/automation where predictable behavior matters
```

So:

> `apt-get` is not deprecated simply because `apt` is more convenient interactively.

For day-to-day terminal use, `apt` is usually clearer. For scripts, `apt-get` remains common because its behavior and output conventions are more suitable for automation.

## Troubleshooting model

When a package-related command fails, separate the problem into layers:

```text
package name / requested operation
        ↓
local package metadata
        ↓
configured repositories
        ↓
network / DNS access
        ↓
package dependency or installation state
```

That prevents treating every package error as the same problem.

## Current level

**Practiced.**

The repository currently claims basic APT workflow knowledge and the `apt`/`apt-get` distinction. It does not claim advanced repository management, pinning, package building, or Debian packaging experience.
