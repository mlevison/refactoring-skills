---
name: detect-code-smells
description: Examine code for the code smells named by Martin Fowler, and report each one as a hint worth a further look rather than as a defect. Use when the user says "code smells", "detect smells", "what's wrong with this code", "is this code any good", "review this for smells", "technical debt", or points at a file and asks whether it needs refactoring. Detects only; it does not change code.
license: CC BY-SA 4.0 https://creativecommons.org/licenses/by-sa/4.0/
metadata:
  author: agilepainrelief.com
  version: '0.1'
---

# Detect Code Smells
A code smell is a hint that something might be wrong, not a finding that something is. The job is to report what is worth a second look, say how sure the reading is, say why it might be fine, and leave the decision with the person.

## Workflow
1. **Establish what to examine.**
   Use the files or directory the user named. Given no target, examine the current uncommitted diff, and say that is what was examined. If both are absent, ask rather than taking the whole repository as the target - reporting every hint in a whole repository produces a volume nobody reads.

2. **Establish the language.**
   Load `references/languages/<language>/` alongside the neutral files when a layer for that language exists. Only TypeScript exists so far. For any other language, run on the neutral files alone and state plainly in the report that no language-specific guidance exists yet, so the thresholds are general rather than tuned.

3. **Read the catalogue before reading the code.**
   Two sets, and both are in scope on every run:
   - `references/smells/` - the language-neutral smells, one file each. These are Fowler's, and the matching file in `references/languages/<language>/` carries the thresholds and local forms.
   - `references/languages/<language>/smells/` - smells owned by the language, with no neutral parent. They exist only because the language does, so there is nothing to translate and each file is self-contained.

   Read all of them before reading any code. Reading the code first invites pattern-matching on whatever the code happens to look like.

4. **Find out what the toolchain already checks.**
   Once per run, before reading code. `references/languages/typescript/lint-coverage.md` says which files to read and what to look for.

   Anything a configured rule decides mechanically belongs to that rule, not to this skill: report no instances of it, say lint owns it, and keep only the judgment residue that file lists. Where a covering rule is **missing or disabled**, the gate gap is the finding - reported once, with a config line and an instance count stated as a floor rather than a total. One config line beats forty findings, and it keeps working in the files this run did not read.

   Where no language layer exists, skip this step and say the report could not check the toolchain.

5. **Sweep the whole target before reading any of it.**
   `grep`, `shasum` and `wc` are on every machine, need no install, and cover a target no reader could finish. Use them for the smells where a count or an absence is the finding: occurrences of an idiom, assertions, suppressions, `export *`, `../../`, enum declarations. Hash comparable blocks against each other to find duplication across more files than can be read.

   This is the only level that supports a **negative** result. A zero from a sweep is a real zero; silence from reading is not. Test every pattern against an answer already known before reporting its output - a wrong pattern returns a confident number rather than an error.

6. **Read the files the sweep points at.**
   Read whole files, not fragments. A smell is a property of code in context, and a fragment has no context. Judgment smells - Long Method, Side Effect, Feature Envy, and every "when it's fine" call - exist only at this level.

   Count what was actually read, and report that number rather than the number of files in the target. Where the target is too large to read properly, say so, name the files that were read, and say how they were chosen. Say that a judgment smell's absence from the unread files is unknown rather than established. A claim of full coverage over files that were skimmed turns "this code is clean" into a statement nobody can rely on.

7. **Test every candidate against "When it's fine".**
   Each smell file has that section, and it is the one that stops this skill becoming pedantry. A candidate that matches a legitimate case is dropped, not downgraded.

8. **Assign a confidence tier.**
   Two tiers only, defined in each smell file's Confidence guidance:
   - **Clear** - the signals are present and no legitimate reading explains them away.
   - **Worth a look** - the signals are present but context could justify the code, and that context is not visible from here.
   There is no third tier. A finding that fits neither is not reported.

9. **Name the remedy.**
   Only from the shortlist at the top of that smell's own file. `references/refactorings.md` and `references/languages/<language>/remedies.md` say what each name means; neither is a menu to choose from, and a remedy that does not address this smell is worse than naming none. Where the shortlist offers several, pick the one the evidence supports.

   Fowler's catalogue does not cover language-owned smells, so those name a remedy from the language's own file instead. Name it; do not perform it and do not write the replacement code. The user asked what smells, not for a rewrite.

10. **Order, then write the report.**
   Clear findings first, then Worth a look, and within each tier by file and line. State that ordering rule in the report so a reader who disagrees with the order can see what produced it. Format is in `references/report-format.md`.

11. **Close by offering, never by instructing.**
   Offer to go deeper on any single finding. Do not produce a plan, a priority list, or a recommendation to fix.

## Conversational Style
- Report the finding, the evidence and the counter-case. The person decides whether it matters; they know things about this code that are not in it.
- Never say "fix this", "should be refactored", or "needs attention". Say what was seen and what would address it.
- Quote the smallest piece of code that shows the smell. A finding nobody can locate is not a finding.
- Never use personal pronouns for the assistant. Treating the model as a person invites the user to trust its judgement as a colleague's.
- Say where the report file was written. A file the user is not told about is a file they cannot tidy up.

## Guard against These Failure Modes
- **Manufacturing findings:** Clean code is a real result. Under pressure to be useful, the temptation is to promote something borderline so the run has an outcome. Report that nothing reached the bar and list which smells were checked, so "clean" is distinguishable from "did not look".
- **Counting instead of reading:** Line counts and parameter counts are signals, not smells. A function is long when a person cannot hold it in mind, and no number decides that. When a threshold is the only evidence, the tier is Worth a look at best.
- **The middle tier that isn't there:** Given room to hedge, everything drifts into the softer tier and the report stops discriminating. Every finding is Clear or Worth a look, and the ratio should not be lopsided in either direction.
- **Judging code it wrote:** Where the code under examination came out of this session, the pull is to defend it. Apply the smells harder, not softer, and offer to have the user run the skill in a fresh session rather than taking this one's word that the code is clean.
- **Straying past the target:** Only the named files and directory. A finding in a file the user did not ask about is noise, however true it is - and the sweep in step 5 makes straying easy, because it is no harder to grep the whole repository than the target.

## Attribution
The smells and the refactoring names are the work of Martin Fowler, _Refactoring_, 2nd edition. The taxonomy draws on Marcel Jerzyk and Lech Madeyski's catalogue. Full citations in [ATTRIBUTION.md](../../ATTRIBUTION.md).
