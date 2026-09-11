# Product Backlog
Ordered most valuable first. Each item records the reasoning, because the reasoning is the part that gets lost.

Not estimates, not story points. This is a list of decisions already argued through, kept so they don't have to be argued again.

---

## A known-answer fixture
Two runs against the YFL expense tracker have corrected real defects, but neither could say what the skill **missed**. A run that reports ten findings and a run that missed forty look identical from outside.

What this needs: a deliberately smelly TypeScript file in the repo, plus a companion file recording exactly which smells are planted where. Then false negatives are measurable, and it becomes a regression test every time a reference file changes - which matters now that reference files are being edited in response to live runs.

### What the runs so far have taught us
Kept because the reasoning is the valuable part, and because it is evidence about the next change too.

- **Two findings named a refactoring that does not address their smell** (`Primitive Obsession → Extract Function`). Cause: the workflow pointed at the whole 32-name catalogue rather than the shortlist on the smell's own file. Fixed, and it is the reason each smell file's shortlist now has to match `refactorings.md` exactly.
- **Zero Side Effect findings across 94 service files.** Not credible. The **When it's fine** section had escape hatches - "it's a command", "the codebase is mutable" - wide enough to excuse an entire service layer. Tightened, and `*Service.ts` is now explicitly not a counter-case.
- **The report claimed 94 files examined with nothing to back it.** No instruction existed to report coverage honestly. Now it reports files *read* against files in target.
- **It found duplication the catalogue had no entry for, and said so rather than mislabelling it.** That behaviour is now specified in the report format, and Duplicated Code was promoted into the catalogue on the strength of it.
- **Wrong target for a first test.** `utils/` and `services/` were many small files, which size-based smells will not trip. Picking a target is part of the test.
- **A missing catalogue entry produces a wrong label, not silence.** Two findings reported as Primitive Obsession in run 1 came back as Duplicated Code in run 3, at identical file and line, once the Duplicated Code entry existed. Primitive Obsession went from three findings to zero. This is the argument for the "Not in the catalogue" report section, now evidenced rather than asserted.
- **The terminal summary dropped findings silently.** A header reading `Clear (12)` above ten rows, and `Worth a look (8) - 0 shown`. The cap was "up to ten, Clear first", which ate two Clear findings with no note - the exact failure the format was written to prevent. Now every Clear finding is listed however many there are, only Worth a look is capped, and a cut tier must say `N shown of M`.
- **Sweeping beats reading for anything countable, and it needs no tooling.** On 651 files of Immich's `web/`, `grep` and `shasum` produced every negative result in the report and its strongest positive finding. Reading 5 files produced the judgment findings and nothing else. Sweep-then-read is now steps 5 and 6 of the workflow.
- **One coverage number for a whole report is misleading.** "4 of 710 files read · 10 findings" let a reader think the other 706 were clean. Coverage is now stated per level - swept, compared, read - with each finding marked, and a judgment smell's absence from unread files called unknown rather than established.
- **The report destination assumed the examined repository was the user's.** Immich's clone is clean and does not gitignore `tmp/`, so following the spec would have left an untracked directory in a third-party checkout. The rule now checks for an existing or gitignored `tmp/` and otherwise **asks** - with an explicit ban on falling back to a system temp directory, where the file would be unfindable and eventually deleted.
- **Strong gates make a short report, and that needs saying out loud.** Immich runs `eslint --max-warnings 0`, `svelte-check --fail-on-warnings` and `tsc --noEmit` in one gating script. Two whole smells reported zero instances because lint owns them. A report that is short for that reason reads identically to a report that is short because nothing was checked, unless it says which.
- **Lint is the better answer wherever a rule is exact.** Raised as an objection and it holds: hand-detecting what `no-floating-promises` decides deterministically is slower, non-deterministic, and does not block a build. Five of the eight TypeScript-native smells were re-scoped to check the *gate* instead of the instances. The dividing line that came out of it is worth keeping: lint wins where the rule is mechanical and exact, reading wins where "when it's fine" decides the answer.

## Structural clone detection
Duplicated Code is now in the catalogue, found by reading. That catches identical and near-identical blocks, which turned out to be most of what a real codebase has.

What reading cannot do: the same algorithm written two different ways, and any comparison at a scale beyond a few files. A detector that hashes and compares does both. Until one is attached, the absence of a duplication finding means nothing was obvious - the smell file and the report both say so, and that honesty is the interim answer.

## Tool-assisted detection, for the tools that need installing
**Reframed by the Immich run.** The original item assumed tool-assisted detection meant adding dependencies - `jscpd`, `ts-morph`, complexity plugins - and was deferred because every one of those is a per-language install that does nothing for the next language.

That premise was half wrong. `grep`, `shasum` and `wc` are on every machine, are language-neutral, and turned out to deliver most of the value: 651 files swept in seconds, five full-coverage negative results, and the single strongest finding of the run (a byte-identical 56-line block across two route files, found by hashing, which no amount of reading would have established). Sweeping is now step 5 of the workflow rather than a backlog item.

What is left in this item is genuinely the part that needs installing: cyclomatic and cognitive complexity, type-aware AST queries, and import-cycle detection. Import cycles are the clearest gap - the Immich run could not check them at all, and had to say so.

No specific tools are recommended yet; surveying what exists and what is still maintained is part of this item.

## Trust the patterns, not just the counts
Two `grep` patterns in the Immich run were wrong and returned confident nonsense rather than errors - one reported 1617 matches where the pattern had collapsed into "any `!` character", its replacement reported 0 against ground truth that was non-zero. The figure finally reported came from a third method, cross-checked by hand against two known instances.

A wrong pattern does not fail. That makes sweeping more dangerous than reading in one specific way: it produces numbers that look authoritative. `report-format.md` now requires testing a pattern against a known answer before reporting its output, but a known-answer fixture would make that mechanical instead of a matter of discipline - which is another reason the fixture item sits at the top.

## React and hooks smells
The fourth category of TypeScript-native smells, deliberately left out of the first batch: lying `useEffect` dependency arrays, state derived into `useState` that should be computed, prop drilling, effects that belong in event handlers.

Held back because it is only worth the files if the codebases under test are React-heavy, and because the three categories shipped - type-system escapes, async hazards, module structure - apply to every TypeScript project regardless of framework. Svelte equivalents would be a separate set again, and the YFL runs suggest that is the more likely need.

## Acknowledged findings
There is no memory between runs. The intended loop is fix five, re-run - and everything not fixed comes back, including the findings already considered and deliberately kept.

That friction grows with every decision made. An ignore file is the obvious answer, and it needs a finding identity that survives the code moving, which is the hard part. In-source suppression comments are ruled out: they put a stale comment in the code and make a read-only skill write to it.

## Rank by blast radius
v1 orders by confidence tier, then file and line. Cheap and stable, but it doesn't distinguish a 200-line function at the heart of the domain from a 200-line seed script.

Ranking by how much depends on the smelly code would put the highest-value work at the top - which matters, because the loop is to fix the top few and re-run. It needs a second analysis pass to find callers, which is the expensive part and the part most likely to be wrong.

## Smells that need change history
**Divergent Change** and **Shotgun Surgery** are invisible to a static read. Both are statements about how code changes over time: many unrelated methods in one class changing together, or one change rippling across many files.

They need git history, not source. That is a different mechanism - probably a different skill - and it is genuinely detectable, unlike guesswork.

## Smells that need whole-program analysis
**Dead Code** and **Speculative Generality** need to know what is reachable. In TypeScript an `export` always looks used from inside its own file, so an LLM reading one file at a time will confidently recommend deleting live code.

This one is dangerous rather than merely noisy, which is why it waits for a tool that can actually resolve references.

## Smells whose OO framing doesn't transfer to TypeScript
Five smells from the glossary need rethinking before they earn a place, because applying them literally to idiomatic TypeScript produces nonsense:

- **Inappropriate Static** - module-level functions are normal in TS. The real analogue is module-level *mutable state*.
- **Refused Bequest** - inheritance is uncommon; structural typing and composition dominate.
- **Null Check** - under `strictNullChecks` most null checks are correct. The analogue is `any` and optional-chaining sprawl.
- **Fate over Action** - TS rarely uses getter/setter pairs. The analogue is exported mutable objects and missing `readonly`.
- **Middle Man** - barrel files and adapters are legitimate patterns here.

Each needs its TypeScript analogue worked out before it can be detected. They stay language-neutral in the catalogue; it is the language layer that has to do the work.

## More languages
The `references/languages/<language>/` split exists so this is additive: write eight files, change nothing else. Python is the obvious next one, being what the source catalogue's own examples use.

Worth doing once, deliberately, after the TypeScript layer has been corrected by real use - so the second language inherits a shape that works rather than the first guess.

## A refactoring skill
`/detect-code-smells` names the refactoring and stops. The other half is performing it - safely, in small steps, with tests run between them.

Kept separate on purpose. A detector that also edits is a detector nobody trusts to run.

## Share one marketplace with agent-thinking-skills
This repo self-hosts a marketplace named `agile-pain-relief-code`, because `agile-pain-relief` is already claimed by [agent-thinking-skills](https://github.com/mlevison/agent-thinking-skills). Two marketplaces from one author is a papercut anyone installing both will hit.

The tidy version is one catalogue listing both plugins. Not urgent - local installs are symlinks and don't touch either marketplace.

---

## Not this repo
**The glossary links to a stale mirror.** [agilepainrelief.com/glossary/code-smells/](https://agilepainrelief.com/glossary/code-smells/) links each smell to `luzkan.github.io/smells`, which is the older Gatsby build of Marcel Jerzyk's catalogue. It still carries an "all rights reserved" footer.

The current home is [codesmells.org](https://www.codesmells.org) - same author, same 56 entries, and MIT licensed. Worth redirecting when the glossary next gets a pass.
