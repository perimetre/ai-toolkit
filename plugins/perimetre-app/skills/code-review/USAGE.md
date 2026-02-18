# Code Review Skill — Usage Guide

## File Structure

```
plugins/perimetre-app/
├── agents/
│   ├── pattern-reviewer.md   ← Sonnet agent, all 11 pattern skills pre-injected
│   ├── bug-scanner.md        ← Haiku agent, shallow diff-only bug scan
│   └── history-reviewer.md   ← Haiku agent, git blame and regression detection
└── skills/code-review/
    ├── SKILL.md       ← Agent skill definition and step-by-step instructions
    ├── PATTERNS.md    ← Detailed pattern examples and anti-patterns
    ├── README.md      ← Overview and quick-start guide
    └── USAGE.md       ← This file
```

## How the Pipeline Works

When `/perimetre-code-review` runs, it executes a 5-step pipeline:

### Step 1: Examine Changes

A Haiku agent runs:

```bash
git status
git diff
git log --oneline -5
```

Returns: which files changed, what patterns they involve.

### Step 2: Independent Review (3 Parallel Agents)

Three agents run in parallel, each covering a different angle:

| Agent | Model | Focus |
|---|---|---|
| `pattern-reviewer` | Sonnet | Audits against all 11 Périmètre patterns (pre-injected at startup via `skills` frontmatter) |
| `bug-scanner` | Haiku | Reads only the git diff, looks for obvious large bugs |
| `history-reviewer` | Haiku | Reads git blame and history, checks for regressions |

The `pattern-reviewer` does not need to read any external files — all 11 Périmètre pattern skills are injected into its context at startup via the `skills` field in its agent frontmatter.

### Step 3: Confidence Scoring

For every issue found, a parallel Haiku agent assigns a confidence score (0–100) using a strict rubric. Agents must double-check that the local pattern documentation explicitly calls out the flagged issue.

### Step 4: Filter

Issues with confidence below 80 are discarded.

### Step 5: Format Output

Remaining issues are formatted as structured markdown: evidence quotes, current code, fix, impact statement, and local reference links.

---

## Invocation Methods

### Slash command (recommended)

```
/perimetre-code-review
```

Runs the full 5-step pipeline using `commands/perimetre-code-review.md`.

### Skill invocation (from other commands/skills)

```markdown
Use the `code-review` skill to review the current changes.
```

Claude will look up `skills/code-review/SKILL.md` and follow its instructions.

### Direct SKILL.md reference

Agents can read `skills/code-review/SKILL.md` directly. Useful when orchestrating from a custom command.

---

## Local Reference Files

All pattern documentation is local — no network requests needed:

| File | Purpose |
|---|---|
| `skills/code-review/PATTERNS.md` | Detailed examples and anti-patterns |
| `skills/error-handling/SKILL.md` | Error-as-values pattern |
| `skills/services/SKILL.md` | Service layer architecture |
| `skills/trpc/SKILL.md` | tRPC routers and procedures |
| `skills/forms/SKILL.md` | React Hook Form + Zod |
| `skills/graphql/SKILL.md` | GraphQL + codegen |
| `skills/tanstack-query/SKILL.md` | TanStack Query patterns |
| `skills/icons/SKILL.md` | @perimetre/icons usage |
| `skills/images/SKILL.md` | Next.js Image component |
| `skills/seo/SKILL.md` | generateMetadata + structured data |
| `skills/caching/SKILL.md` | Caching and ISR |
| `skills/payload-drizzle/SKILL.md` | Payload CMS + Drizzle ORM |

---

## Confidence Score Rubric

Scoring agents use this rubric verbatim:

| Score | Meaning |
|---|---|
| **0** | False positive or pre-existing issue not introduced in this diff |
| **25** | Might be real but unverified; stylistic concern not explicitly documented |
| **50** | Real issue but minor nitpick; limited practical impact |
| **75** | Verified real issue that will be hit in practice; directly documented |
| **100** | Definite issue confirmed by evidence; will happen frequently |

Issues scoring below 80 are filtered out before output.

---

## Sharing with Your Team

This skill is distributed as part of the `perimetre-app` Claude Code plugin. To give your team access:

1. Install the `perimetre-app` plugin in Claude Code's plugin marketplace
2. All commands, skills, and agents — including `/perimetre-code-review` — become available automatically

See the plugin root `README.md` for installation instructions.
