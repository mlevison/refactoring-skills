# Sweeping
The sweep answers three different kinds of question, and each has a different tool. Picking the wrong one is how a pattern comes back with a confident number instead of an error.

| The question is about | Tool | Why not the others |
| --- | --- | --- |
| **Shape** - a call with a particular argument, a member chain of a given depth, a statement nested inside two others, a parameter with a given type annotation | `ast-grep`, where it is installed | Regex cannot see structure. Every collapsed pattern this skill has shipped was a structural question asked lexically. |
| **Text** - does this string appear, how often, in which files | `grep` | `ast-grep` offers nothing on a question about characters, and costs a rule file to ask it. Import paths, comment markers, `export *`. |
| **Whole-file properties** - length, and block-for-block identity | `wc`, `shasum` | Neither of the other two can count lines or hash a block. |

## ast-grep
Probe once per run with `command -v ast-grep`. Invoke it as `ast-grep`, never as `sg`: the short alias collides with shadow-utils' `sg` on Linux.

It parses rather than matches characters, so it is the right tool for the structural half of Message Chain, Long Parameter List, Flag Argument, Conditional Complexity nesting, and the naming rule in Side Effect - all of which regex asks badly.

What it cannot do, in any version:
- **Types.** Tree-sitter parse only. Whether `order` is an `Order`, or a promise is unawaited, is outside it permanently. That half belongs to lint.
- **Cross-file anything.** Rules run per file. Every threshold phrased "in 2 or more places" is outside what a rule can say.
- **Counting.** Rules match; they do not tally. Thresholds need `ast-grep scan --json` piped to something that counts.

## Languages it does not cover
`.ts` and `.tsx` are built in. `.svelte` and `.vue` are **not**, and making them work needs either a `languageGlobs` mapping to HTML or a compiled tree-sitter library registered in `sgconfig.yml`.

Do not write config into the target repository to get there. Where the target is Svelte or Vue, sweep those files with `grep` and say in the report that the structural sweeps did not cover them.

## The known-answer check applies to both
A wrong `ast-grep` pattern returns `0` exactly as confidently as a wrong regex. Parsing buys accuracy, not self-validation. Test every pattern, in either language, against an answer already known before reporting its output.

## What the report has to say
Absent the binary, say so once: the structural sweeps were approximations and their counts are floors rather than totals. Naming the tool that was used is what lets a reader tell a real zero from a pattern that never matched.
