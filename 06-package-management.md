# 6. Debian/Ubuntu Package Management

This section documents the APT package-management workflow I have practiced on Debian/Ubuntu systems.

## What I know

APT separates package metadata from the packages already installed on the machine.

```text
configured repositories
        ↓
apt update
        ↓ refresh local package metadata
available package versions
        ↓
apt upgrade / apt install / apt remove
        ↓
changes to installed packages
```

That distinction matters because:

```bash
sudo apt update
```

does **not** install every available upgrade. It refreshes the local package index so APT knows what versions the configured repositories currently provide.

## What I can do with APT

### Refresh package information

```bash
sudo apt update
```

Use this when I need current repository metadata before searching, installing, or upgrading packages.

### Upgrade installed packages

```bash
sudo apt upgrade
```

This uses the refreshed metadata to upgrade installed packages where the normal upgrade can be completed without removing installed packages.

The relationship is:

```text
apt update  → learn what versions are available
apt upgrade → apply available upgrades to installed packages
```

### Install and remove packages

```bash
sudo apt install <package>
sudo apt remove <package>
```

`install` resolves and installs the requested package and required dependencies. `remove` removes the package while normally leaving package configuration files that may be useful for a later reinstall.

### Search and inspect before installing

```bash
apt search <term>
apt show <package>
```

I use them for different questions:

```text
apt search → what package names/descriptions match this term?
apt show   → what is this specific package, version, dependency set, and description?
```

This lets me inspect package metadata before changing the system.

## `apt` versus `apt-get`

I understand `apt` as the newer interactive command intended to combine common package-management operations into a friendlier interface.

`apt-get` is the older interface that remains common in scripts and automation because its command-line behavior is designed to be more stable for that use.

For normal interactive administration, I practiced commands such as:

```bash
apt update
apt upgrade
apt install
apt remove
apt search
apt show
```

The important distinction is not that one command is universally “better.” It is that they are interfaces to the same underlying package-management system with somewhat different intended use cases.

## How package symptoms narrow my first question

```text
"package not found"         → is package metadata current, and is the repository configured?
"which package matches X?"  → apt search, then apt show the likely package
"updates are available"     → update metadata first, then decide whether to upgrade
"need this tool installed"  → inspect/search package, then install
```

## What this lets me do

I can reason about package management as a state transition rather than memorize isolated commands:

```text
repository metadata
→ refresh local index
→ inspect available package
→ install/upgrade/remove
→ verify the tool/package state afterward
```
