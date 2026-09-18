# Product Backlog
Ordered most valuable first. Each item keeps the reasoning behind it, since that's the part that gets lost.

Not estimates, not story points - decisions already argued through, kept so they don't have to be argued again.

---

## A known-answer fixture
Two runs so far have corrected real defects, but neither could say what the skill **missed** - a run with ten findings and a run that missed forty look identical from outside.

Needs a deliberately smelly TypeScript file plus a companion file naming which smells are planted where. That makes false negatives measurable and gives a regression test for every reference-file edit - which matters now that reference files change in response to live runs.

## Structural clone detection
Duplicated Code is in the catalogue, found by reading - which catches identical and near-identical blocks, most of what a real codebase has.

Reading can't catch the same algorithm written two different ways, or any comparison beyond a few files. A hash-and-compare detector does both. Until one is attached, no duplication finding means nothing was obvious, not that nothing exists - the smell file and report both say so.

## Tool-assisted detection, for the tools that need installing
**Reframed by the Immich run.** Assumed tool-assisted detection meant adding dependencies (`jscpd`, `ts-morph`, complexity plugins) and was deferred as a per-language install that helps no other language.

Half wrong: `grep`, `shasum` and `wc` are on every machine, language-neutral, and delivered most of the value - 651 files swept in seconds, five full-coverage negative results, and the run's strongest finding (a byte-identical 56-line block across two route files, found by hashing alone). Sweeping is now step 5 of the workflow, not a backlog item.

What's still left to install: cyclomatic and cognitive complexity, type-aware AST queries, and import-cycle detection - the clearest gap, since the Immich run couldn't check it at all.

No tools recommended yet; surveying what's maintained is part of this item.

## Trust the patterns, not just the counts
Two `grep` patterns in the Immich run returned confident nonsense instead of errors: one matched 1617 times after collapsing into "any `!` character"; its fix matched 0 against known-nonzero ground truth. The number finally reported came from a third method, hand-checked against two known instances.

A wrong pattern doesn't fail - it produces numbers that look authoritative, which makes sweeping more dangerous than reading. `report-format.md` now requires testing a pattern against a known answer first. A known-answer fixture would make that mechanical instead of a matter of discipline - another reason that item sits at the top.

## React and hooks smells
The fourth TypeScript-native category, deliberately left out of the first batch: lying `useEffect` dependency arrays, state that should be computed rather than held in `useState`, prop drilling, effects that belong in event handlers.

Held back because it only pays off on React-heavy codebases, while the three shipped categories - type-system escapes, async hazards, module structure - apply to every TypeScript project regardless of framework. Svelte equivalents would need a separate set again, and real-world runs so far suggest that's the more likely need.

## Acknowledged findings
No memory between runs. The loop is fix five, re-run - so everything not fixed comes back, including findings already considered and deliberately kept.

That friction compounds with every decision made. An ignore file is the obvious fix, but needs a finding identity that survives the code moving - the hard part. In-source suppression comments are ruled out: they leave a stale comment behind and make a read-only skill write to code.

## Rank by blast radius
v1 orders by confidence tier, then file and line - cheap and stable, but it can't tell a 200-line function at the heart of the domain from a 200-line seed script.

Ranking by how much depends on the smelly code would put the highest-value work first, which matters since the loop is fix-top-few-and-re-run. Needs a second pass to find callers - the expensive part, and the part most likely to be wrong.

## Smells that need change history
**Divergent Change** and **Shotgun Surgery** are invisible to a static read - both describe how code changes over time: many unrelated methods in one class changing together, or one change rippling across many files.

They need git history, not source - a different mechanism, probably a different skill, and genuinely detectable rather than guesswork.

## Smells that need whole-program analysis
**Dead Code** and **Speculative Generality** need to know what's reachable. An `export` always looks used from inside its own file, so an LLM reading one file at a time will confidently recommend deleting live code.

Dangerous rather than noisy - waits for a tool that can actually resolve references.

## Smells whose OO framing doesn't transfer to TypeScript
Five smells from the glossary need rethinking before they earn a place, because applying them literally to idiomatic TypeScript produces nonsense:

- **Inappropriate Static** - module-level functions are normal in TS. The real analogue is module-level *mutable state*.
- **Refused Bequest** - inheritance is uncommon; structural typing and composition dominate.
- **Null Check** - under `strictNullChecks` most null checks are correct. The analogue is `any` and optional-chaining sprawl.
- **Fate over Action** - TS rarely uses getter/setter pairs. The analogue is exported mutable objects and missing `readonly`.
- **Middle Man** - barrel files and adapters are legitimate patterns here.

Each needs its TypeScript analogue worked out before it's detectable. They stay in the language-neutral catalogue; the language layer does the work.

## More languages
The `references/languages/<language>/` split makes this additive: write a new layer, change nothing else. `references/languages/typescript/README.md` is the shape to copy - manifest plus recommended tools and adoption order. Python is the obvious next one, since the source catalogue's own examples use it.

Worth doing once, deliberately, after real use has corrected the TypeScript layer - so the second language inherits a shape that works, not a first guess.

## A refactoring skill
`/detect-code-smells` names the refactoring and stops. The other half is performing it - safely, in small steps, with tests run between them.

Kept separate on purpose. A detector that also edits is a detector nobody trusts to run.

## Share one marketplace with agent-thinking-skills
This repo self-hosts a marketplace named `agile-pain-relief-code`, because `agile-pain-relief` is already claimed by [agent-thinking-skills](https://github.com/mlevison/agent-thinking-skills). Two marketplaces from one author is a papercut anyone installing both will hit.

The tidy version is one catalogue listing both plugins. Not urgent - local installs are symlinks and don't touch either marketplace.

---

## Not this repo
**The glossary links to a stale mirror.** [agilepainrelief.com/glossary/code-smells/](https://agilepainrelief.com/glossary/code-smells/) links each smell to `luzkan.github.io/smells`, the older Gatsby build of Marcel Jerzyk's catalogue. It still carries an "all rights reserved" footer.

The current home is [codesmells.org](https://www.codesmells.org) - same author, same 56 entries, MIT licensed. Worth redirecting when the glossary next gets a pass.
