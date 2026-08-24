# Text, Log, and Data Processing

## Purpose

Linux troubleshooting often starts with text: logs, configuration files, command output, CSV data, or process information. The useful skill is not knowing one command in isolation; it is building a pipeline that turns noisy input into the specific fact needed for the investigation.

## Reusable pipeline model

```text
raw input
   ↓
select relevant rows
   ↓
extract relevant fields
   ↓
normalize / sort
   ↓
count / aggregate / calculate
   ↓
inspect the result
```

## Searching with `grep`

`grep` was used for filtering text and searching file contents.

Basic pattern:

```bash
grep "pattern" file
```

Counting matching lines:

```bash
grep -c "pattern" file
```

Regular-expression anchors also came up during `/proc` investigation. For example:

```bash
grep '^secret:' file
```

Here:

```text
^
→ beginning of the line
```

So the pattern means "match `secret:` only when it begins the line."

## Field processing with `awk`

`awk` was used when the problem was based on columns rather than whole lines.

For access-log analysis, the first field could be extracted with a pattern like:

```bash
awk '{print $1}' access.log
```

For numeric processing, values can be accumulated:

```awk
sum += $2
```

The important concept is that `$1`, `$2`, and so on represent fields in the current record.

## Counting repeated values

A recurring Unix pattern is:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

Conceptually:

```text
extract value
   ↓
sort identical values together
   ↓
count each group
   ↓
rank counts
   ↓
inspect the highest result
```

This pattern was used during access-log analysis where the most frequent IP occurred **482 times**.

The transferable lesson is that `uniq -c` only counts adjacent duplicates, which is why `sort` normally comes first when the original data is not already grouped.

## Arithmetic and precision

A separate exercise required calculating an average from the second column of a file and controlling decimal precision.

The logic was:

```text
sum column 2
   ↓
count rows
   ↓
divide total by row count
   ↓
control decimal output
```

`awk` handled field processing and accumulation, while `bc` was used to reason about decimal arithmetic and `scale`.

Example concept:

```bash
echo 'scale=2; 10/3' | bc
```

The main lesson was that output requirements such as "exactly two decimal places" or truncation are part of the problem, not a cosmetic afterthought.

## Narrowing output

Commands such as:

```bash
head
tail
```

were used to reduce large command output to the portion that mattered.

`tail -f` also served a different purpose: observing a file while another process continued writing to it.

## CSV and structured-text practice

Completed hands-on exercises also included:

- breaking/transforming a CSV file
- merging multiple CSV files

The exact solution commands for those exercises are not reconstructed here because the retained command trail is incomplete. They are counted as completed practice in [the practice log](../practice-log.md), while this page only documents command patterns that can be supported confidently.

## Command-line investigation

The Command Line Murders exercise reinforced a broader pattern:

```text
large set of text clues
        ↓
search for names / attributes / line references
        ↓
filter candidate information
        ↓
combine multiple clues
```

That type of work is useful because real troubleshooting rarely asks for a command by name. It asks a question and requires choosing the right filters.

## Current level

**Practiced.**

Strongest current areas:

- `grep`
- `awk`
- `sort`
- `uniq -c`
- simple pipelines
- field extraction
- frequency counting
- basic arithmetic over text data

Further repetition will focus on more complex `awk`, `sed`, shell scripting, and structured formats beyond basic CSV/text.
