# 1. File and Text Investigation

This section documents the file, text-processing, and shell-pipeline skills I can use as building blocks for larger Linux investigations.

## What I know

I have hands-on practice with:

```text
find      → locate filesystem objects
cat       → read a known file
head/tail → narrow output
grep      → search text
awk/cut   → extract fields
sort      → order values
uniq -c   → count adjacent repeated values
md5sum    → verify file contents by hash
|         → send one command's output into another command
>         → redirect output into a file
```

I also understand that `/proc` is not an ordinary directory tree on disk. It is a virtual filesystem exposing live kernel and process state through file-like paths.

## What I can do with those tools

### Locate a file and then inspect its contents

If I know a file exists somewhere under a path, but I also need the file whose contents match a condition, those are two separate questions.

```bash
find /proc/sys -type f
find /proc/sys -type f -exec grep -l '^secret:' {} +
```

The logic is:

```text
find → locate candidate files
        ↓
grep → test their contents
```

`^secret:` means the line must begin with `secret:`. `-type f` keeps the search to regular files, and `-exec ... {} +` passes groups of found paths to `grep`.

That gives me a reusable distinction:

```text
Where is the object? → find
What does it contain? → grep
```

### Turn a log into a frequency count

If an access log begins each record with a source IP and I need the most frequent source, I can build the answer in stages:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

What each stage contributes:

```text
awk '{print $1}' → extract the IP field
sort              → group identical values together
uniq -c           → count each grouped value
sort -nr          → rank counts from highest to lowest
head               → show the top result
```

In one lab dataset, the top source appeared 482 times. The count answered the frequency question; it did not by itself prove anything malicious about that source.

From this I can recognize that wording such as **“most common,” “most frequent,” or “appears the most”** usually calls for some variation of:

```text
extract → group → count → rank
```

### Calculate from structured text

If the required value comes from a numeric column, I can use `awk` to accumulate the field and then divide by the record count.

```awk
sum += $2
```

That means: for every input row, add field 2 to the running value `sum`.

For an average, I break the problem into separate checks:

```text
correct column?
→ correct running total?
→ correct number of rows?
→ correct division?
→ correct decimal precision?
```

I used `bc` when decimal precision needed to be controlled explicitly.

### Choose tools from the question instead of memorizing a pipeline

The wording of the task often tells me what kind of operation I need:

```text
"find the file" / "where is" → find
"contains" / "starts with"   → grep
"first/second column"         → awk / cut
"how many matches"            → grep -c
"how many of each value"      → sort + uniq -c
"most common"                 → extract + sort + uniq -c + rank
"average of a column"         → sum + count + divide
```

The exact command still depends on the data, but this narrows the choice to the tool that exposes the fact I need.

## How this connects to later troubleshooting

The same inspection pattern carries into other Linux subsystems:

```text
file contents       → cat / grep
process information → ps
socket information  → ss / fuser
service state       → systemctl
historical logs     → journalctl
```

The object being inspected changes, but the reasoning stays similar:

```text
What information do I need?
→ which tool exposes it?
→ which part of the output answers the question?
→ what should I inspect next?
```
