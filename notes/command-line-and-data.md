# Command Line and Data Work

This note collects the command-line patterns practiced across the Linux exercises instead of splitting each small topic into its own document.

## APT basics

Practiced package-management operations:

```bash
sudo apt update
sudo apt upgrade
sudo apt install <package>
sudo apt remove <package>
apt search <term>
apt show <package>
```

The key distinction is:

```text
apt update  -> refresh available package metadata
apt upgrade -> upgrade installed packages
```

`apt` is the friendlier interactive interface. `apt-get` remains valid and is still commonly used in scripts because its behavior is more stable for automation.

## Text and log processing

Frequently used tools:

```bash
grep
awk
sort
uniq
head
tail
```

A useful access-log pattern is:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

That turns raw log entries into a ranked frequency count. In the Saskatoon exercise, the most frequent IP appeared **482 times**.

`grep` was also used for content searches and exact-pattern matching. For example:

```bash
grep '^secret:' file
```

Here `^` means the beginning of the line.

## Field processing and arithmetic

`awk` was used to work with column-based data, including extracting fields and accumulating numeric values:

```awk
sum += $2
```

`bc` was used when decimal arithmetic and output precision mattered.

The general pattern was:

```text
extract fields -> aggregate/count -> calculate -> format result
```

## CSV and structured-text exercises

Completed practice also included transforming a CSV file and merging multiple CSV files. The exact command trails for those exercises are not reconstructed because they are not retained accurately enough to present as evidence.

## What this demonstrates

- basic Debian/Ubuntu package management
- `grep` and regular-expression anchors
- `awk` field processing
- Unix pipelines
- sorting and frequency counting
- basic arithmetic over text data
- CSV/text manipulation

This is hands-on training work, not professional Linux administration.