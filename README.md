# Assess (Feasibility & Effort)

A Claude Code plugin that answers **feasibility and effort questions** about a codebase — *"is it possible to…", "what's the effort for…", "how hard would it be to…"* — through codebase exploration and collaborative clarification.

It explores the relevant services, asks clarifying questions one at a time, proposes 2-3 approaches with effort estimates, and presents a structured assessment. Then it **stops** — it never designs or implements. When you're ready to build, it hands you off to `superpowers:brainstorming`.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code)
- [Superpowers plugin](https://github.com/obra/superpowers) (recommended — for the `brainstorming` handoff when you're ready to build)

## Installation

**Install via VS Code (recommended):**

1. Open the Claude Code panel in VS Code
2. Type `/plugins` in the chat
3. Go to **Marketplaces** tab → enter `leoawesome/assess` → add it
4. Go to **Plugins** tab → find **assess** → click **Install**
5. Done — works in every session

**Install via CLI:**

```
/plugin marketplace add leoawesome/assess
/plugin install assess@assess-marketplace
/reload-plugins
```

**Private repos:** This works with private repos too. As long as you have `git clone` access (e.g., added as a collaborator), Claude Code uses your existing git credentials. For auto-updates on private repos, set `GITHUB_TOKEN` in your shell config.

**For local testing:**

```bash
claude --plugin-dir /path/to/assess/plugin
```

## Usage

Ask a feasibility or effort question — the skill activates automatically on phrases like:

- "Is it possible to add multiple assignees to a contact?"
- "What's the effort for exporting reports to CSV?"
- "How hard would it be to support webhooks?"
- "Does the platform support X?"

It will **not** activate for implementation requests ("build X", "implement X") — those go to `superpowers:brainstorming` — or for simple lookups ("where is X", "how does Y work"), which it answers directly.

## How It Works

The skill follows a fixed four-step flow, and the **terminal state is a written assessment** — never code or a design doc:

1. **Explore project context** — identifies the relevant services and dispatches exploration subagents to understand the current data model, existing patterns, and what already exists.
2. **Ask clarifying questions** — one at a time, informed by what it found in the codebase, preferring multiple-choice questions shaped by real findings.
3. **Propose 2-3 approaches** — each with trade-offs, the services it would touch, and an effort estimate (Small / Medium / Large / Very Large).
4. **Present the assessment** — a structured summary of what exists today, the recommended approach, what would change, the effort estimate, and verified risks. Then it stops.

A hard gate prevents the skill from writing code, creating design docs, or invoking implementation skills. If you want to proceed, it points you to `superpowers:brainstorming`.

## Effort Scale

| Size | Range | Examples |
|---|---|---|
| **Small** | 1-2 days | config change, new field on existing model, new route using existing patterns |
| **Medium** | 3-5 days | new model/table, new queue flow, changes across 2-3 services |
| **Large** | 1-2 weeks | new subsystem, changes across 4+ services, new infrastructure |
| **Very large** | 2+ weeks | architectural change, new service, cross-cutting concerns |

## Customization

The skill is a single self-contained file: `plugin/skills/assess/SKILL.md`. To adapt it to your codebase:

- **Codebase map** — the exploration step references a project conventions doc (e.g. a Product Modules table in `service/CLAUDE.md`). Point this at your own project's module map.
- **Effort scale** — tune the day/week ranges to match your team's velocity.
- **Dimensions to cover** — edit the clarifying-question dimensions table to fit your domain.

## License

MIT
