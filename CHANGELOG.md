# Changelog

## 0.1.0
First release.

- `detect-code-smells` skill: reads a file, a directory, or the current diff and reports Fowler's smells plus TypeScript-native ones, each with evidence, a confidence tier, a "when it's fine" case, and the remedy that would address it. Detects only; changes nothing.
- Language-neutral catalogue under `references/smells/`, with a TypeScript layer under `references/languages/typescript/` for thresholds and local forms.
- Installable via `npx skills`, the Claude Code plugin marketplace (`agile-pain-relief-skills`), or by hand.
