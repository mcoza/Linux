# 1. File and Text Investigation

This stage developed the ability to extract useful evidence from files, logs, and command output. The important distinction was not simply knowing commands, but selecting a tool based on whether I needed to locate an object, search its contents, extract fields, count values, or transform data.

## Locating objects versus searching contents

**Situation:** I needed to identify which file in a directory tree contained a particular value.

```bash
find /path -type f
grep '^pattern:' file
find /path -type f -exec grep -l '^pattern:' {} +
```

**Why these commands:** `find` locates filesystem objects using properties such as path and type. `grep` searches text inside those objects. Combining them answers a different question: which regular file under this path contains the required content?

**Interpretation:** The `^` anchor restricts the match to the beginning of a line. `-type f` avoids treating directories and other object types as ordinary files.

**Applied context:** I used this relationship while searching under `/proc/sys`. Because `/proc` exposes live kernel and process state rather than ordinary disk files, permission errors and changing entries had to be interpreted as part of the environment.

**Next step:** Inspect the matching file and confirm that the value—not merely the filename—answers the original question.

## Turning an access log into a frequency count

**Situation:** A raw access log contained many source addresses, and I needed to determine which occurred most frequently.

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

**Why this pipeline:**

- `awk` extracts the relevant field.
- `sort` groups identical values.
- `uniq -c` counts each group.
- `sort -nr` ranks the numeric counts.
- `head` limits the result to the highest entries.

**Interpretation:** Each program performs one transformation. The value of the pipeline comes from passing progressively refined output to the next command. In the applied exercise, the highest-frequency address appeared 482 times.

**Next step:** Decide whether the frequency is expected traffic, an operational anomaly, or security-relevant evidence; the count alone does not establish intent.

## Column-based arithmetic

**Situation:** I needed to calculate a value from a numeric column and control decimal output.

```awk
sum += $2
```

`awk` was appropriate because the input was field-oriented. `bc` was used when shell arithmetic was insufficient for decimal calculations and output precision mattered.

**Reasoning:** The task could be decomposed into extracting the correct field, accumulating the values, dividing by the correct record count, and formatting the result. Verifying each intermediate value made an incorrect final answer easier to trace.

## Additional applied work

I also completed exercises involving searching and filtering evidence across text files, transforming a CSV file, and merging multiple CSV files.

Those exercises reinforced the same pattern:

```text
locate input → select fields → filter/transform → aggregate → verify output
```

The exact command trails for the CSV exercises were not retained accurately enough to present as reproducible walkthroughs.

## What changed in my troubleshooting approach

I moved from viewing `grep`, `find`, `awk`, `sort`, and `uniq` as isolated commands to treating them as tools for successively narrowing raw system data into evidence that answers a specific question.
