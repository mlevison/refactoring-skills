# Report Format
Two outputs from every run: a full report on disk, and a short summary in the terminal. The file holds everything found. The terminal holds enough to act on, plus an honest count of what it left out.

## The File
Named `code-smells-YYYY-MM-DD-HHMM.md`, or `code-smells-<target>-YYYY-MM-DD.md` where that reads better.

**Where it goes depends on whether the examined repository is the user's to write to.** Work down this list and stop at the first that holds:

1. **`tmp/` already exists at the repository root** - write there.
2. **`tmp/` is absent but gitignored** (`/tmp/`, `tmp/` or a pattern covering it appears in `.gitignore`) - create it and write there. The ignore rule is the repository saying this is the scratch location.
3. **Neither** - **ask the user where to put it.** Offer the two obvious candidates: create `tmp/` at the repository root anyway, or write somewhere outside the repository that they name. Say why it is being asked: creating an un-ignored directory leaves an untracked path in their working tree, and in a repository they do not own it is a change they did not request.

Do not guess, and do not fall back on a system temporary directory - a file there cannot be found, remembered or tidied up by the user, and the operating system eventually deletes it, so the work is silently lost.

Whichever path is used, **say where the file was written** in the terminal output. A file the user is not told about is a file they cannot tidy up.

Never write findings into the source files as comments. This is a read-only operation, and a suppression comment is a comment that goes stale.

```markdown
# Code Smells - <what was examined>
<date and time> · <n> files read of <n> in target · <n> findings

Ordered: Clear findings first, then Worth a look, and within each tier by file and line.
A smell is a hint that a further look is warranted, not a defect.

<When no language layer exists: "No TypeScript/<language> guidance exists in this catalogue
yet, so the guidance below is general rather than tuned to this language.">

## Clear

### <Smell Name> - `path/to/file.ts:42`

<The smallest quotation of code that shows it.>

**What was seen** - the specific signals in this code, not the definition of the smell.

**Why it might be fine** - the strongest case that this code is right as it stands. Never
omit this, and never write a token version of it. If no such case exists, say so and say why.

**What would address it** - <Refactoring Name>. One sentence. No replacement code.

## Worth a look

<Same structure.>

## Gate gaps
<Only where a covering rule is missing or disabled. One entry per rule, never per instance.
Name the rule, the config line, and the instance count reading found - stated as a floor,
because the rule will find the rest including in files this run did not read.

Where the rules are all in place, say so in one line. It is the most reassuring thing in the
report, and it explains why whole smells are absent from it.>

## Not in the catalogue

<Only when something real has no entry. Describe it, locate it, count it, and say plainly
that the catalogue has no name for it. Do not file it under the nearest smell - a
mislabelled finding is worse than an unnamed one, because the label stops anyone looking
further. These entries are the best evidence of what the catalogue is missing.>

## What was checked
<Every smell read in step 3, listed by name - the neutral ones and the language-owned ones.>
Smells requiring change history or whole-program analysis are outside what this skill
examines - see the product backlog.

<Where reading found less than the whole target: which files were read, and how they were
chosen.>
```

## The Terminal
**Every Clear finding is listed, however many there are.** Only *Worth a look* is capped, at six. A tier that is cut says so on its own header, as `N shown of M`, and a tier that is complete says nothing.

Never print a tier count that disagrees with the number of rows beneath it. If the header says 12 and 10 rows follow, two findings have been lost and the report is now worse than no report. Where the Clear list is long enough to be unreadable, that is information about the code - print it anyway and let the length say so.

```
Code smells - src/billing/ · 6 of 6 files read · 14 findings

Clear (4)
  Long Method            src/billing/invoice.ts:88    → Extract Function
  Flag Argument          src/billing/invoice.ts:12    → Remove Flag Argument
  Duplicated Code        src/billing/lines.ts:40      → Extract Function
  Side Effect            src/billing/tax.ts:19        → Separate Query from Modifier

Worth a look (10) - 6 shown of 10
  Primitive Obsession    src/billing/money.ts:4       → Replace Primitive with Object
  ...

Full report with the reasoning and the counter-case for each: tmp/code-smells-2026-09-10-1642.md
```

Where fewer files were read than the target holds, say how they were chosen on the first line of the terminal output too, not only in the file. A reader who sees `156 of 326` and no basis for the 156 has no way to weigh the result.

## When Nothing Reaches the Bar
Say so, and say what was examined. Silence is indistinguishable from a run that failed.

```
Code smells - src/billing/ · 6 of 6 files read · nothing reached the reporting bar

Checked: Long Method, Long Parameter List, Flag Argument, Conditional Complexity,
Primitive Obsession, Message Chain, Feature Envy, Side Effect, Duplicated Code.
TypeScript: Any Leakage, Unsafe Assertion, Suppressed Diagnostic, Floating Promise,
Sequential Await, Barrel File Coupling, Deep Relative Import, Enum Over Union.

Not examined: smells needing change history (Divergent Change, Shotgun Surgery) or
whole-program analysis (Dead Code, Speculative Generality). Duplicated Code was read
rather than computed, so it finds textual clones and misses structural ones.
```

No report file is written in this case, and nothing is promoted to fill the space.

## On Counts
Every count in a report has to be one that was taken, not one that sounds right. Files read, occurrences of a duplicated block, instances of an idiom - state the number only where it was counted, and say "several" where it was not. A fabricated count is the fastest way to make an accurate report untrustworthy.

**A mechanical count is only as good as the pattern behind it.** Test the pattern against an answer already known - a line quoted earlier in the same run - before reporting its output. A wrong pattern does not fail; it returns a confident number. Two plausible-looking patterns for the same thing disagreeing by orders of magnitude is the normal case, not a freak one.

## On Coverage
One coverage number for the whole report is misleading, because different findings rest on different reach. State it per level, and mark findings accordingly.

- **Swept** - a mechanical pass with `grep`, `shasum` or similar over every file in the target. Cheap, complete, and the only thing that can support a **negative** result: a zero here means zero. Counts and absences belong to this level.
- **Compared** - blocks or files hashed against each other. Finds exact and near-exact duplication across a set no reader could hold at once.
- **Read** - files read in full. The only level that can support Long Method, Side Effect, Feature Envy or any "when it's fine" judgment, and always the smallest.

Say which level produced each finding, and say plainly that a judgment smell's absence from the unread files is **unknown rather than established**. Reading 5 files of 651 and reporting "9 findings" invites the reader to think the other 646 were clean.

Sweep before reading. It is faster, it covers everything, and it tells you which files are worth reading.
