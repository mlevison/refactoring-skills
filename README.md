# Refactoring Skills
Agent/Claude Skills for finding code smells and, in time, for refactoring them away.

> A **code smell** is a hint that something might be wrong in your codebase. A smell isn't always a problem, rather it's a hint that a further look is warranted. Code smells are a clue that says more time is needed to examine this area of the code.
>
> — [Agile Pain Relief glossary: Code Smells](https://agilepainrelief.com/glossary/code-smells/)

That distinction is the whole design. A tool that reports smells as defects gets switched off within a week, because most of what it finds is code somebody had a reason for. So every finding here arrives with the strongest case *against* itself, and the decision stays with the person reading it.

More important than ever in the era of AI-assisted coding, where the volume of plausible-looking code has gone up and the number of people who have read it has not.

## The Skills
| Skill | What it's for |
| --- | --- |
| [Detect Code Smells](skills/detect-code-smells/) | Reads a file, a directory or your current diff and reports nine of Fowler's smells plus eight TypeScript-native ones - each with the evidence, a confidence tier, why the code might be fine, and the remedy that would address it. Detects only; changes nothing. |

## What To Ask
Point it at code, or at nothing:

- `/detect-code-smells src/billing/`
- `/detect-code-smells` — examines your uncommitted diff
- "Are there code smells in the invoice module?"
- "Claude wrote this yesterday, what smells?"

## Languages
Language-neutral by design, TypeScript first because that is what needed refactoring.

The catalogue is split in two: `references/smells/` holds the ideas, which do not change between languages, and `references/languages/<language>/` holds the thresholds and the local forms, which do. A 30-line function means something different in TypeScript than in Python, and the number belongs where that is true.

Some smells have no neutral form at all. A floating promise, an `any` leaking inward, a cycle through a barrel file - these exist because TypeScript exists, so they live entirely under `references/languages/typescript/smells/` with nothing to translate. Fowler's catalogue has no refactoring for them either, so they name a remedy from the language's own list.

Run against a language with no layer yet, the skill uses the neutral files and says so, rather than pretending its thresholds were tuned for your code.

## Roadmap
The [product backlog](docs/product-backlog.md) has the detail and the reasoning. In short: tool-assisted detection, Duplicated Code, the smells that need change history or whole-program analysis, more languages, and eventually a skill that performs the refactorings rather than naming them.

## A Note on Language
These skills never have the model refer to itself as a person. The same rule governs the instructions inside each `SKILL.md`: they say what **the assistant** should do rather than addressing it as "you", so nothing in the text implies there is someone in there being spoken to.

A model that speaks as "I" invites you to treat it as a colleague who has weighed the evidence and formed a view. On a skill that reports judgements about your code, that matters more than usual - the findings are hints, and hints need checking.

This convention is shared with [Agent Thinking Skills](https://github.com/mlevison/agent-thinking-skills).

## Installing a Skill
### npx skills
```bash
npx skills add mlevison/refactoring-skills                          # asks where to put it
npx skills add mlevison/refactoring-skills -g                       # personal, every project
npx skills update                                                   # pull later versions
```

### Claude Code plugin
```
/plugin marketplace add mlevison/agile-pain-relief-skills
/plugin install refactoring-skills@agile-pain-relief-skills
```

The first line registers the catalogue and installs nothing. The second does the installing. [agile-pain-relief-skills](https://github.com/mlevison/agile-pain-relief-skills) is one marketplace for all of Agile Pain Relief's skills, so the same catalogue also offers [Agent Thinking Skills](https://github.com/mlevison/agent-thinking-skills) — add it once, install whichever plugins you want.

To remove: `/plugin uninstall refactoring-skills@agile-pain-relief-skills`.

This repository used to be its own marketplace. If you installed that way, `/plugin uninstall refactoring-skills@agile-pain-relief-code` and `/plugin marketplace remove agile-pain-relief-code` first.

### By hand
Each skill is a self-contained directory under `skills/`. Copy the whole thing, `references/` and all, or the skill won't load:

- **Claude Code (project)** - into `.claude/skills/`, committed
- **Claude Code (personal)** - into `~/.claude/skills/`, for every project
- **Claude apps** - zip the directory and upload it

Symlinking a clone instead of copying makes `git pull` the update:
`ln -s "$PWD/skills/detect-code-smells" ~/.claude/skills/detect-code-smells`

Anthropic's guide: https://support.claude.com/en/articles/12512180-using-skills-in-claude

## Repository Layout
```
.claude-plugin/
  plugin.json       # this repo as a single plugin; the catalogue that
                    # lists it is github.com/mlevison/agile-pain-relief-skills
docs/
  product-backlog.md
skills/
  <skill-name>/
    SKILL.md        # frontmatter + workflow; what Claude loads
    README.md       # the human-facing explanation
    references/     # catalogue and detail, loaded on demand
```

`SKILL.md` stays short on purpose. Anything long lives in `references/` so it's only read when it's actually needed.

## Related
[Agent Thinking Skills](https://github.com/mlevison/agent-thinking-skills) - Systems Thinking, Critical Thinking, and Critical Thinking for GenAI. Same house, different problem: those skills question your reasoning, these ones read your code.

## GenAI Usage
Claude is used to help design the skills and write the installation instructions. The core content remains human authored.

## Contributing
Issues first, please, for ideas every bit as much as for bugs. It's the cheapest place to find out whether something fits.

Smell definitions are argued over more than most text. A **When it's fine** section that is wrong makes the skill nag, and a threshold that is wrong makes it a line counter - so changes to those are worth a conversation before anyone spends an evening on them.

Pull requests written by an agent won't be merged.

## License
[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) - full text in [LICENSE](LICENSE).

Copy, adapt and build on these skills, commercially or otherwise. Two conditions: keep the attribution, and put the same licence on what you build.

That holds for derivative work assembled by a GenAI agent from these files exactly as it holds for work assembled by hand. The obligation sits with whoever directed the agent. Keep this line:

> Mark Levison, [Agile Pain Relief](https://agilepainrelief.com) - [github.com/mlevison/refactoring-skills](https://github.com/mlevison/refactoring-skills)

[ATTRIBUTION.md](ATTRIBUTION.md) says the same thing in a form an agent can parse, and carries the citations for the sources this catalogue builds on.

Created by [Agile Pain Relief](https://agilepainrelief.com)
