# Design: assess skill enhancements

**Date:** 2026-06-03
**Target:** `plugin/skills/assess/SKILL.md`
**Scope:** Five localized prose edits. No new sections, no flowchart changes. ~143 → ~175 lines.

## Background

The `assess` skill is a feasibility/effort-assessment skill scoped to the respond.io
`rocketbots` monorepo. A multi-lens review (7 lenses, 37 raw findings, 24 kept after
adversarial YAGNI vetting) found the skill fundamentally sound but surfaced one
robustness gap and four bounded sharpenings. The skill stays deliberately lean —
nothing here adds a section or a scoring framework, and the explicitly rejected
items (persisted `ASSESSMENT.md`, fenced ClickUp template, numeric scoring rubric,
rename to dodge `assess:assess`) are recorded below so they are not re-litigated.

## Changes

### 1. Resilient orientation (`CLAUDE.md`)

**Why:** The first exploration instruction pointed at "the **Product Modules** table
in `service/CLAUDE.md`". That heading does not exist (the real content is the
**Directory Structure** tables + **Technology Stack** section), and the file is
branch-fragile — on a branch where it isn't present (or when cwd is a subdirectory),
the most load-bearing orientation step dead-ends.

**Edit (line 64):** Replace the single bullet with a primary + fallback + warn chain:
- Primary: orient from `service/CLAUDE.md` — its Directory Structure tables (ECS
  services + Lambda functions) and Technology Stack section are the module map and
  data-layer guide. If cwd is a subdirectory, search upward for the nearest `CLAUDE.md`.
- Fallback (absent): list `service/*/` and `lambda/*/` to enumerate modules, skim the
  relevant service's `README`/`package.json`, and tell the user `CLAUDE.md` wasn't
  found so they can check their branch — orientation is best-effort without it.

This absorbs two earlier findings ("add the lambda layer to scope", "add the
RDS/DynamoDB/OpenSearch tri-store to scope") — both are already documented in
`CLAUDE.md`, so the fix is to lean on it properly rather than hand-duplicate them.

### 2. Activation boundary

**Why:** The frontmatter description's only negative guard was "not implementation".
A one-line lookup ("does the platform support webhooks?") still auto-fires the full
clarification ritual, because the lookup guard lived only in the body (read after the
skill is chosen).

**Edits:**
- Rewrite the description (line 3) to carry BOTH exclusions — implementation/design of
  something already decided → `superpowers:brainstorming`; factual questions about
  existing code → answer directly — plus the "doesn't exist yet" scope cue and the
  "what would it take to" / "roughly how long" phrasings.
- Reconcile the Trigger Patterns block (lines 22–29): drop the lookup-flavored
  "Does the system support..." from the activate list, add a **capability-question**
  disambiguation rule (answer-directly if it exists; assess only if it doesn't), and
  add a **decision-state** rule (already decided + wants design help → brainstorming).

### 3. Process branches (comparison + multi-feature)

**Why:** The flow is a single linear spine assuming one feature, refined and estimated.
Two common shapes break it: comparison ("which of these 3 is cheapest?") and
multi-feature requests (the skill says "flag immediately" then dead-ends).

**Edit (after line 76):** Add two conditional bullets in "Understanding the requirement":
- Multiple independent features → after flagging, offer (a) assess one in depth now, or
  (b) one-line coarse band per feature to sequence the work + recommend an order
  (explicitly a sequencing aid, not a final assessment).
- Comparing candidates → ask only the 1–2 questions that change the *relative* ranking,
  coarse-band each, present a ranked table (candidate | effort | main cost driver |
  recommendation); full 2–3-approaches treatment only if the top two are close.

### 4. Effort calibration + confidence + spike

**Why:** The scale anchors purely on coding scope and presents every estimate with
identical certainty.

**Edits:**
- After the effort scale (line 109): add "estimates cover the full delivery slice"
  framing + "what bumps it up a bucket" cost-driver cues (large migration/backfill;
  queue/message-format change; cross-service contract/public-API change; high-traffic
  critical path; **searchable/filterable field on a contact-like entity → touches the
  OpenSearch index, so Medium not Small**; unclosable unknown). Plus a one-line
  familiarity caveat (ranges assume an engineer already familiar with the service).
- Add the "**When an unknown blocks an honest estimate**" branch (spike-gated /
  external-dependency): name the deciding question + who/what answers it, give a
  conditional estimate, recommend a spike as next step (not implementation).
- Effort estimate line (124): add "**Spike required** (conditional estimate)" as a
  valid value and a **confidence level** (High/Med/Low) tied to verified-in-code vs
  assumed; Low confidence → span across two buckets ("Medium-to-Large").

**Dropped as unverified:** the claim that DB migrations live in a separate
`respond-io/migrations` repo. `CLAUDE.md` documents migrations in each service's
`database/` dir, so only the searchable-field → OpenSearch caveat is kept.

### 5. Context-carrying handoff

**Why:** The terminal handoff is a bare "Start with `superpowers:brainstorming`", but
brainstorming re-runs explore-context and clarify from scratch, discarding the
assessment's main output.

**Edit (line 132):** Extend the handoff so brainstorming reuses the assessment (current
state, chosen approach, services to touch, clarified requirement) as the answers to its
explore-context and clarify steps, instead of re-deriving from zero.

## Explicitly NOT doing (YAGNI)

- No persisted `ASSESSMENT.md` artifact and no "we deliberately don't persist" clause.
- No copy-paste fenced ClickUp template (output is already paste-ready markdown).
- No numeric scoring rubric / mandated subagent counts / expanded dimensions table.
- No rename of `assess:assess` (collision risk with `modernize-assess`/review skills is
  negligible — those are command-only or diff-framed).
- No flowchart changes — the new branches are conditional prose only.
- Keep the named "I Can Assess This After One Question" anti-pattern section standalone.

## Shipping

The skill now lives in the `leoawesome/assess` plugin repo (installed at project scope
as `assess@assess`). Shipping = edit `plugin/skills/assess/SKILL.md`, bump the version
in `plugin.json` + `.claude-plugin/marketplace.json`, push, then
`claude plugin update assess@assess` + reload.
