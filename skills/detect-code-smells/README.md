# Detect Code Smells
Part of [Refactoring Skills](../../README.md).

Point it at a file, a directory, or nothing at all, and it reports what smells - with how sure it is, why the code might be fine as it stands, and the name of the refactoring that would address it.

It reads. It does not edit.

## The Problem This Solves
Two failure modes, opposite to each other, and most tools pick one.

A linter counts. It tells you a function is 47 lines without knowing whether those 47 lines are one idea or four, so it is wrong often enough that people turn it off.

An LLM asked "is this code good?" agrees with whatever you seem to want, or manufactures a finding because producing nothing feels unhelpful. Under pressure to be useful, it promotes something borderline so the run has an outcome.

This skill is built against both. Every smell has a **When it's fine** section that drops candidates rather than downgrading them, and every finding has to carry the strongest counter-case alongside the evidence. Clean code is a reportable result, and the skill says which smells it checked so you can tell "clean" from "didn't look".

## What You Get
Nine of Fowler's smells, all detectable by reading the code in front of you:

| Smell | The question it asks |
| --- | --- |
| **Long Method** | Can a person hold this function in mind at once? |
| **Long Parameter List** | Is there an unnamed concept in this signature? |
| **Flag Argument** | Does `true` mean anything at the call site? |
| **Conditional Complexity** | Has anyone named the question this condition asks? |
| **Primitive Obsession** | Is the same rule about this string enforced in four places? |
| **Message Chain** | Was the caller made to learn an object graph to get one value? |
| **Feature Envy** | Does this logic live away from the data it works on? |
| **Side Effect** | Does the function do more than its name promises? |
| **Duplicated Code** | Is this idea written down more than once? |

### Where lint is the better answer
Eight more smells exist only because TypeScript does. For most of them **a lint rule does the job better than reading** - deterministically, across every file, in CI where it blocks the build. So the skill checks the *gate* rather than listing instances.

Where a covering rule is enabled, it reports nothing and says lint owns it. Where the rule is missing or disabled, the missing rule is the finding: one config line, with the instance count as a floor rather than a total. One rule beats forty findings, and it keeps working in the files nobody read.

The single highest-value check it makes: `typescript-eslint`'s plain `recommended` config does **not** enable the type-aware rules, so a repo on `recommended` has `no-floating-promises` and the whole `no-unsafe-*` family switched off and usually doesn't know it.

What's left after lint is the judgment no rule can make — how far an `any` travelled from its boundary, a type predicate whose body checks less than its signature claims, an import pointing in the wrong direction:

| TypeScript smell | The question it asks |
| --- | --- |
| **Any Leakage** | How far did this `any` travel from the boundary that made it? |
| **Unsafe Assertion** | What breaks if this `as` or `!` is wrong? |
| **Suppressed Diagnostic** | What error is this comment hiding, and how would anyone know it was fixed? |
| **Floating Promise** | Who finds out if this fails? |
| **Sequential Await** | Does iteration *n* actually need iteration *n-1*? |
| **Barrel File Coupling** | Is there a cycle through this `index.ts`? |
| **Deep Relative Import** | Does this path encode the filesystem into the source? |
| **Enum Over Union** | Is the runtime enum object used at all? |

**Why the Fowler smells above aren't treated the same way.** ESLint has `complexity`, `max-params`, `max-lines-per-function` and `sonarjs/no-identical-functions`, which cover crude versions of four of them. Those rules are exactly what this skill exists to improve on: `max-lines-per-function` cannot tell a flat data mapping from four tangled responsibilities, and that judgment is the entire product. Set them as a cheap mechanical floor if you like. They are not a substitute for reading, and this skill does not check whether you have them.

Each finding gives you the code, what was seen, **why it might be fine**, and the remedy that addresses it - named, not performed. Fowler's smells name a refactoring from his catalogue; the TypeScript-native ones name a remedy from `references/languages/typescript/remedies.md`, since Fowler's catalogue does not cover them.

When it finds something real that the catalogue has no name for, it says so in its own section rather than filing it under the nearest smell. Those entries are how the catalogue grows - Duplicated Code was added because a run reported it that way.

Two confidence tiers, deliberately. **Clear** means the signals are there and nothing explains them away. **Worth a look** means context could justify it and that context isn't visible from here. There is no third tier, because a middle bucket is where everything ends up when hedging is available.

## When to Use It
- Before refactoring, to find out where to start
- On a pull request, alongside a review rather than instead of one
- On code a model generated, which is where these smells now arrive fastest
- On unfamiliar code, as a way of reading it

**Trigger phrases:** "code smells", "detect smells", "what's wrong with this code", "review this for smells", "does this need refactoring"

Given no target, it examines your uncommitted diff.

## What It Won't Do
It won't change your code. Not the smells, not the comments, not a suppression marker.

It won't sweep your whole repository. A repo-wide run produces hundreds of hints, which is a wall nobody reads and the fastest way to make the skill worthless.

It won't tell you to fix anything. A smell is a hint, and whether it matters depends on things that aren't in the code - how often this file changes, who maintains it, whether it ships next week. You know those. It doesn't.

It won't find every smell. Some need change history (Divergent Change, Shotgun Surgery) and some need whole-program analysis (Dead Code, Speculative Generality). Those are named in the [product backlog](../../docs/product-backlog.md) rather than guessed at.

Duplicated Code is a partial answer rather than a missing one. Reading finds copy-paste and near-copy-paste, which is most of what a real codebase has. It does not find the same algorithm written two different ways, and it cannot scan at a scale where a clone detector would. The smell file says so, and so does the report.

And it has no memory between runs. A finding you looked at and decided was fine comes back next time. That friction is real and it is logged.

## What's in Here
- `SKILL.md` - the workflow Claude follows
- `references/smells/` - one file per neutral smell: what it is, what it costs, the mechanical and judgment signals, when it's fine, and how to pick a confidence tier
- `references/languages/typescript/README.md` - what the TypeScript layer holds, and which tools to recommend in which order
- `references/languages/typescript/<smell>.md` - the same nine smells with actual numbers, TypeScript-specific forms, and the idioms that only look like smells
- `references/languages/typescript/smells/` - the eight TypeScript-owned smells, self-contained because there is no neutral parent to translate
- `references/refactorings.md` - Fowler 2nd edition names, with 1st edition aliases
- `references/languages/typescript/remedies.md` - fixes for the TypeScript-native smells
- `references/languages/typescript/lint-coverage.md` - which rule owns which smell, what each rule cannot decide, and the config worth recommending when nothing is set up. The skill reads your config; it never runs a linter.
- `references/sweeping.md` - which sweep tool answers which kind of question, and what `ast-grep` cannot do
- `references/report-format.md` - the shape of the report and the terminal summary

The split between `smells/` and `languages/` is deliberate, and it holds in both directions. Thresholds are language-dependent while the ideas are not, so a neutral smell has a thin language layer. A smell that exists *because* of a language has no neutral form at all, so it lives entirely under that language. Adding Python means adding `references/languages/python/`, not forking the catalogue.

## Installation
See [Installing a Skill](../../README.md#installing-a-skill) in the repository README.

In Claude Code, invoke it with `/detect-code-smells`, or let Claude reach for it when a trigger phrase turns up.

## Credit
The smells and the refactoring names are Martin Fowler's, from _Refactoring_, 2nd edition. The taxonomy draws on Marcel Jerzyk and Lech Madeyski's [catalogue](https://www.codesmells.org). The framing - a smell is a hint, not a defect - comes from the [Agile Pain Relief glossary](https://agilepainrelief.com/glossary/code-smells/). Full citations in [ATTRIBUTION.md](../../ATTRIBUTION.md).

## License
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) - attribution terms in the [repository README](../../README.md#license).

Created by [Agile Pain Relief](https://agilepainrelief.com)
