# WorkDesk OS — Five-Zone Vault Plan

## Context

This plan crystallizes the five-zone WorkDesk OS architecture so it can be reviewed by Ultra Review / Codex before implementation begins. The vault has lived in a four-zone model (personal, atlas, intel, system) since vault-architecture; testing with Jenny Meier and the Codex POBO review revealed that GTD-style action management was scattered across project notes, meeting notes, and inline checkboxes with no canonical home. Splitting GTD into its own zone resolves the tension and gives each zone a single unit type to manage.

The output of this plan is a working WorkDesk OS bootstrap that any user can install on a fresh machine, scaffold a starter vault, and use immediately — with Claude proactively extending the vault over time via four meta-skills and a self-improvement loop.

## Five-Zone Model

Each zone manages one unit type. Each zone has one job.

| Zone | Unit | Job | Agent writes? |
|---|---|---|---|
| `personal/` | practice | Practice management (journal, daily, reading) | No — read-only |
| `atlas/` | object | Object management — single-source identified | Yes |
| `gtd/` | action | Action management — projects, actions, inbox | Yes |
| `intel/` | signal | Signal management — multi-source gathering | Yes |
| `system/` | source | Source management — raw inputs + infrastructure | Yes (hooks) |

### Atlas vs Intel — the purpose test

Both zones can hold notes informed by multiple sources. The split is **purpose**, not source count.

| Zone | Asks | How sources work |
|---|---|---|
| **Atlas** | *"Did this happen / does this exist?"* — evidence | Multiple sources allowed; every claim cites its source (per source-documentation rule) |
| **Intel** | *"What does this mean for me?"* — synthesis | Multiple sources, filtered through Claude's operator-context lens; regenerable |

- **Atlas** = factual record of operator's life, work, relationships, decisions. WHAT.
- **Intel** = Claude's interpretation through operator-context. WHAT IT MEANS.

Examples:
- Meeting note records what was said → atlas
- Observation interpreting what 5 meetings reveal as a pattern → intel
- Person note accumulates evidence over time, every claim sourced → atlas
- Daily plan synthesizes today's priorities from many sources → intel

**The deeper test:**
- **Atlas** asks: *"Is this a factual record of something that happened or exists?"*
- **Intel** asks: *"Is this Claude's interpretation through operator-context?"*

Atlas notes that accumulate over time (people, companies, engagement statuses) remain atlas — they record evidence with claim-level source attribution. Intel is regenerable; atlas is not.

## Zone Plans

*(Each zone is filled in as we walk through it together.)*

### Zone 1: Personal — locked

**Unit:** practice. **Job:** practice management. **Agent writes?** Never.

**Structure:** practices are user-driven folders. No fixed defaults.

```
personal/
  daily/             # ships pre-built (the only default practice)
  {user-defined}/    # added over time via /define-practice
```

**Hard-locked read-only.** A `PreToolUse` hook in `.claude/settings.json` blocks Write, Edit, and NotebookEdit operations whose path resolves under `personal/`. Enforcement is at the harness level — not just instruction. Reads (Read, Glob, Grep) are allowed.

**Practice declaration** at `.claude/practices/{name}.md`: identity, cadence, template, read policy, detection clause.

**Pre-built declarations shipped with bootstrap:**
- `daily` — daily note practice (cadence: daily; template: minimal; read-policy: morning daily-plan reads today's note)

**`/define-practice` meta-skill** scaffolds new practices through a JTBD-first interview. Onboarding may propose common ones (journal, reading) but never auto-creates them.

**Migration:** out of scope for this bootstrap. The existing `personal/journal/`, `personal/notes/`, `personal/books/`, `personal/identity/` content stays where it is in Khalil's current vault. The fresh bootstrap delivers only the read-only lock + daily practice. Migration happens in a separate session after operator validates the bootstrap.

### Zone 2: Atlas — locked

**Unit:** object. **Job:** object management — single-source identified evidence. **Agent writes?** Yes.

**Source rule:** atlas notes carry claim-level source attribution per the source-documentation rule. A note may aggregate multiple sources over time (a person note accumulates from many meetings; an engagement status updates session after session) — but every claim cites where it came from. Atlas answers "did this happen / does this exist?" Intel asks "what does this mean for me?" Multi-source is fine in both zones; the split is purpose.

**Folder per object type. Universal-four ship; engagement containers come from onboarding; everything else is emergent:**

```
atlas/
  meetings/        # universal — ships pre-built — single notes
  decisions/       # universal — ships pre-built — single notes
  people/          # universal — ships pre-built — single notes
  initiatives/     # universal — ships pre-built — folders, 8-item structure each
  {engagement-containers}/  # added during onboarding — see below
  {emergent-types}/         # added later via vault-improvements signal proposals
```

**Engagement containers — the one onboarding question.** Onboarding asks: *"What kinds of long-running contexts do you have?"* and offers persona-keyed prompts. Operator can rename freely; same `/define-object` machinery underneath.

| Persona prompt | Resulting container | Lighter 4-item folder per instance |
|---|---|---|
| *"Consultant?"* | `clients/` | `_brief.md`, `_status.md`, `notes/`, `_archive/` |
| *"Founder/owner?"* | `businesses/` | same |
| *"Employee at an organization?"* | `teams/` or `departments/` | same |
| *"Researcher/academic?"* | `collaborations/` or `labs/` | same |
| *"Creative work?"* | `disciplines/` (or by user-named) | same |
| *"Personal use?"* | `areas/` (life areas) | same |

**Emergent objects (NOT in onboarding — proposed by vault-improvements signal over time):**

| Object type | Trigger for proposal |
|---|---|
| `companies/` | Repeated org references across meetings/transcripts |
| `content/` | Content-production patterns or explicit "draft" / "publish" notes |
| Anything else (deals, listings, book-notes, grants, etc.) | Pattern detection via vault-improvements |

**Two folder shapes for atlas objects:**
1. **Atomic objects** (meetings, decisions, people, companies-when-emergent) → single notes per instance
2. **Container objects with ongoing context** (initiatives, engagements like clients/businesses/teams/labs/etc., future deals/listings) → folders with their own minimum structure

The object declaration says which shape applies.

**Engagement folders — 4-item structure (regardless of what they're called):**

```
atlas/{engagement-type}/{slug}/
  _brief.md       # who/what they are, how you work together
  _status.md      # current state of the relationship/context
  notes/          # ongoing observations
  _archive/       # retired material
```

NO nested `projects/` subfolder. Engagement-tied work lives flat in `atlas/initiatives/` linked back via `engagement: "[[atlas/{engagement-type}/{slug}]]"` frontmatter. Engagement `_status.md` references its active initiatives.

**Initiatives — 8-item structure (same as projects):**

```
atlas/initiatives/{slug}/
  _brief.md, _status.md, plan.md, notes/, reference/, specs/, deliverables/, _archive/
```

Frontmatter: `engagement: "[[atlas/clients/dudley]]"` or `engagement: "[[atlas/businesses/benali]]"`.

**Object declaration** at `.claude/objects/{type}.md` carries:
- **Identity** — name, folder location, single-note vs folder, file-naming pattern
- **Format** (not template) — frontmatter fields + body sections Claude follows
- **Source** — required field on every instance; accepts wikilinks to logged sessions OR processed materials
- **Detection** — when Claude proposes creating a new instance during processing
- **Matching** — what else updates when this changes (per the matching rule)
- **Lifecycle** — status values, transitions, archive rules

**Universal frontmatter baseline (every object instance):**

```yaml
---
type: <object-type>
status: active | archived | <type-specific>
source: "[[...]]"           # required — points to logged session or processed material
created: 2026-04-26
last_updated: 2026-04-26
author: claude | operator
---
```

Every `/define-object` declaration extends this with type-specific fields.

**Detection → GTD inbox.** When Claude proposes creating a new object during processing, it drops a `[REVIEW]` pointer into `gtd/inbox/`. Operator confirms before scaffolding fires.

**Content lives across zones (not locked to atlas):**
- **Content artifact** (single-source piece) → `atlas/content/` (created via `/define-object` if user produces content)
- **Claude-generated content from multi-source synthesis** → `intel/signals/`
- **Personal writing** (operator's own drafts, pre-processing) → `personal/{practice}/`
- **Content production work** (research → draft → publish — multi-session) → `gtd/projects/{slug}/`, deliverables land in `atlas/content/` when shipped

**The action vs project vs initiative test (universal):**
- *Single-session execution by Claude or operator* → **action** (`gtd/actions/next/`), can link to engagement directly
- *Multi-session operator attention required, no engagement* → **project** (`gtd/projects/`)
- *Multi-session engagement-tied work* → **initiative** (`atlas/initiatives/`)

Tasks do not exist as a separate unit. Anything multi-session warrants the 8-item structure (project or initiative). Anything single-session is an action — even if it has multiple sub-checkboxes inline.

**Migration:** out of scope for this bootstrap. Existing `atlas/projects/`, `atlas/clients/{slug}/projects/`, `atlas/businesses/{slug}/projects/` content stays where it is in Khalil's current vault. Bootstrap delivers the new model on a fresh machine. Migration handled in a separate session after operator validates the bootstrap.

### Zone 3: GTD — locked

**Unit:** action. **Job:** action management — projects, actions, inbox triage. **Agent writes?** Yes.

**Ships whole** with the bootstrap. NOT extensible via `/define-X`. GTD is the same for every user because GTD is a known framework.

**Folder structure:**

```
gtd/
  inbox/              # review queue — pointers to items needing operator attention
  actions/
    next/             # physical next actions ready to do
    waiting/          # delegated / awaiting another human
  projects/           # personal focus list (small) — multi-session operator attention
    {slug}/           # 8-item structure each (9-item if code project)
  someday/
    actions/          # parked single-step items
    projects/         # parked multi-step outcomes (full folder if applicable)
  archive/
    projects/{year}/{slug}/    # full project folder, intact
    actions/{year}-{month}/    # archived actions, by month
```

**Status changes = file moves.** `next/ → waiting/ → archive/`. Agent moves files programmatically; operator drags in Obsidian. No Bases required.

**Action format:**

```yaml
---
type: action
status: next                                          # next | waiting | someday | done
context: [work, calls, errands]
parent: "[[atlas/initiatives/dudley-msa-review]]"     # polymorphic — points to whatever owns this action: gtd project, atlas initiative, engagement, or empty
waiting-on: "[[atlas/people/...]]"                    # only if status: waiting
source: "[[atlas/meetings/2026-04-22-dudley-weekly]]" # logged session OR processed material
created: 2026-04-26
---
What needs to happen and why.
```

**`parent:` is polymorphic.** One field, points to whatever owns the action. Claude dereferences the link to figure out the parent's type. Examples:
- `parent: "[[gtd/projects/workdesk-os]]"` — owned by a personal-focus project
- `parent: "[[atlas/initiatives/dudley-msa-review]]"` — owned by an engagement-tied initiative
- `parent: "[[atlas/clients/dudley]]"` — tied directly to an engagement, no project/initiative wrapping
- `parent: ""` — standalone

Single field works automatically for new container types added later (e.g., user-defined object types). No schema updates needed.

**Project format (gtd/projects/{slug}/) — 8-item per per-project-accounting rule:**

```
_brief.md       # Purpose, Principles, Outcome, Vision (POBO output)
_status.md      # Current phase, next action, open items, last_updated
plan.md         # POBO plan snapshot — phases + intended steps
notes/          # running captures
reference/      # source material, research, inputs
specs/          # detailed specs for builds
deliverables/   # final outputs
_archive/       # retired material inside the project
```

(9th item `repo/` for code projects.)

**Inbox format:**

```yaml
---
type: inbox-item
prefix: REVIEW                                         # REVIEW | ACTION | QUESTION | CONTENT | AWARENESS
target: "[[atlas/decisions/2026-04-23-dudley-scope-change]]"
source: "[[atlas/meetings/2026-04-23-dudley-weekly]]"
created: 2026-04-26
---
One-line description of why this needs review.
```

**Inbox rules:**
- Only Claude writes inbox items
- Only operator clears them
- Pointers, not content (the real note lives in atlas or intel; inbox just points)
- `[AWARENESS]` items auto-expire after N days

**POBO → project flow:** POBO produces `_brief.md`, `_status.md`, `plan.md`. The plan holds intended phase steps as text. **Only the current physical next action gets promoted** to `gtd/actions/next/` as an action object. Future steps stay in `plan.md` until promotion. Promotion is always agent-proposed, operator-confirmed.

**Action vs project test (the AI-era reframe):**
- *Single-session execution by Claude or operator* → **action** in `gtd/actions/next/`, can `parent:` an engagement directly
- *Multi-session operator attention required* → **project** in `gtd/projects/` (personal focus) OR **initiative** in `atlas/initiatives/` (engagement-tied)
- The test is universal: *am I going to be engaged across multiple sessions?*

**Pre-built declarations shipped with bootstrap:**
- `.claude/objects/action.md` — action format + lifecycle
- `.claude/objects/project.md` — project format + scan locations (`gtd/projects/`, `atlas/initiatives/`)
- `.claude/objects/inbox-item.md` — inbox format + prefix rules

### Zone 4: Intel — locked

**Unit:** signal. **Job:** signal management — Claude's interpretation through operator-context. **Agent writes?** Yes.

**Folder structure — explicit folder per signal type:**

```
intel/
  briefings/
    daily/                # daily plan — high cadence
      2026-04-26-daily-plan.md
      _archive/2026-03/   # archived after 30 days
    weekly/               # weekly review — low cadence, no archive
  observations/           # patterns Claude surfaces
    _archive/{year-month}/  (when it earns it)
  vault-improvements/     # the self-improvement loop — no archive (history valuable)
  research/               # external reads + synthesized concepts — no archive (knowledge)
  {user-defined}/         # added via /define-signal
```

**Universal signal frontmatter:**

```yaml
---
type: signal
shape: briefing | observation | research | vault-improvement | <user-defined>
date: 2026-04-26
sources: ["[[...]]", "[[...]]"]
schedule: daily | weekly | on-demand | triggered
---
```

**Signal declaration** at `.claude/signals/{type}.md`:
- **Identity** — name, folder location, file-naming pattern
- **Sources** — what it gathers from (calendar, email, atlas, log.md, etc.)
- **Format** — frontmatter + body sections Claude follows
- **Schedule** — on-demand, daily, weekly, event-triggered
- **Output** — where it lands; whether it drops a `[REVIEW]` inbox pointer
- **Detection** — when Claude proposes generating one ad-hoc

**Pre-built signals shipped with bootstrap:**

**1. `daily-plan`** (`.claude/signals/daily-plan.md`)
- Sources: today's calendar, unread email, today's daily note, yesterday's meeting transcripts, recent log.md, active gtd/projects/ statuses, active atlas/initiatives/ statuses
- Schedule: daily (morning trigger or on-demand via `/daily-ops`)
- Output: `intel/briefings/daily/{date}-daily-plan.md`

**2. `vault-improvements`** (`.claude/signals/vault-improvements.md`)
- Sources: `system/log.md` event stream, unused atlas folders, stale projects, inbox backlog, signals not opened, captures sitting unprocessed
- Schedule: weekly
- Output: `intel/vault-improvements/{date}-vault-improvements.md` + drops `[REVIEW]` inbox pointers for each recommendation
- Holistic scan across atlas, gtd, intel, personal, system — whole-vault, not per-zone
- The self-improvement loop. Ships pre-built because it's what makes the vault evolve over time.

**`/define-signal` meta-skill** scaffolds new signals via JTBD-first interview:
- *"What do you want Claude to keep an eye on for you?"*
- *"What sources should it look at?"*
- *"How often should it run?"*
- *"What should it produce?"*

Meta-skill writes `.claude/signals/{name}.md`, creates `intel/{name}/` folder, optionally schedules.

**Archive policy by cadence:**

| Signal type | Archive after |
|---|---|
| Daily plan, daily observations | 30 days → `_archive/{year-month}/` |
| Weekly review | never (low volume) |
| Vault-improvements | never (history valuable) |
| Research / reads | never (knowledge) |
| One-off observations | never |

Archived signals stay in the vault, fully readable, just out of the active scan path. vault-improvements and other long-horizon analyses still scan `_archive/` when needed.

### Zone 5: System — locked

**Unit:** source. **Job:** source management — raw inputs + activity infrastructure. **Agent writes?** Yes (mostly via hooks).

**Folder structure:**

```
system/
  intake/              # generic raw drops awaiting triage (replaces both inbox/ and _processing/)
  transcripts/         # raw meeting transcripts before processing
  bookmarks/           # raw saved articles before processing (only if user has Defuddle/Keep)
  session-log/         # per-session narratives, written by /extract
    2026-04-26-{slug}.md
  event-log.md         # append-only event stream — every vault operation, one line each
  event-log-archive/
    2026-03.md         # rolling monthly archive
```

**Raw sources persist after processing.** Transcripts, bookmarks, captures stay in their folder — never deleted. Frontmatter flips `processed: true` and `processed-into: ["[[...]]"]` lists the atlas/intel notes that came from it. Per-source-type rule: `move-after-processing: false | "_archive/{year-month}/"`. Default = false (keep in main folder).

**Universal source frontmatter:**

```yaml
---
type: source
source-kind: transcript | bookmark | session-log | intake | other
date: 2026-04-26
processed: false                # flips to true after atlas/intel extraction
processed-into: ["[[...]]"]    # backlinks to notes produced from this source
---
```

**Source declaration** at `.claude/sources/{type}.md`:
- **Identity** — name, folder location, file pattern
- **Format** — what raw input looks like
- **Processing rule** — how it becomes atlas/intel/gtd notes
- **Retention** — keep forever, archive after N days, delete after N days

**Pre-built source declarations (universal):**
- `transcript` — Granola/Google Meet/manual transcripts. Processing rule: extract meeting → `atlas/meetings/` + decisions → `atlas/decisions/` + people updates
- `session-log` — claude-log entries. Written by `/extract` at session end. Format: summary section (wikilink-able) + full conversation appended verbatim
- `intake` — generic raw drops. Processing rule: triage to atlas/intel/gtd

**Conditional source declarations (added later):**
- `bookmark` — only if user installs Defuddle/Keep.md (proposed by vault-improvements when first article URL appears in intake/)

**Session-log shape** — narrative + full conversation in one file:

```markdown
---
type: session-log
date: 2026-04-26
duration: ~90 min
---

# Summary
[3-5 sentences. Wikilink-able. What happened, decided, changed.]

# Conversation
[Verbatim input/output, every turn, in order. Referenceable for "what did we actually say?"]
```

`/extract` writes the summary manually for readability. The full conversation is appended automatically (captured during the session).

**event-log.md mechanism (hook-driven, no token cost to write):**

A `PostToolUse` hook in `.claude/settings.json` fires after every Write/Edit/Bash that touches the vault. The hook script:
1. Inspects the tool call (file path, before/after, command)
2. Categorizes the operation (object created, action moved, signal generated, transcript processed)
3. Appends one line to `system/event-log.md` in format: `YYYY-MM-DD HH:MM | operation | target | result`

**Reading is windowed.** Session start reads last 7-14 days only. Older entries roll into `system/event-log-archive/{YYYY-MM}.md` automatically (monthly rotation hook).

**Daily summary lines** at end of day compact a day's events into one entry: `DAY-SUMMARY | 47 ops, 3 meetings, 5 actions done`. Signals scan summaries first, drill into detail only as needed.

**session-log/ vs event-log.md:**
- `session-log/` = per-session narrative ("what we discussed and decided this session"). Written by `/extract`.
- `event-log.md` = per-operation event stream. Written by hooks.
- A single session typically produces 1 entry in `session-log/` and dozens in `event-log.md`.

**Reference / scripts / templates moved to `.claude/`:**

These are infrastructure, not sources. `system/` is now purely sources + activity logs.

```
.claude/
  scripts/           # hook scripts, automation
  templates/         # reusable scaffolds for meta-skills
  hooks/             # hook definitions referenced from settings.json
```

SOPs and workflows are NOT shipped in V1. Future `/define-sop` or `/define-workflow` meta-skills if users ask. For now, `.claude/reference/` doesn't exist.

**`.claude/` visibility in Obsidian — symlink approach:**

Bootstrap creates a symlink at vault root: `_workdesk/` → `.claude/`. Obsidian shows `_workdesk/` in the file tree (underscore sorts to top); all configurations visible. Claude Code still finds `.claude/` natively because the actual folder exists. Editing `_workdesk/{file}` IS editing `.claude/{file}` — same underlying file, two paths.

Windows fallback: install the "Show Hidden Files" Obsidian plugin (Windows symlinks require admin/developer mode, which is friction).

## Cross-Cutting Concerns

### Six meta-skills (final set)

1. **`/define-object`** — atlas content types
2. **`/define-signal`** — intel signal types
3. **`/define-source`** — system source types
4. **`/define-practice`** — personal practice types
5. **`/define-tool`** — Claude capability/integration (CLI, API, MCP)
6. **`/define-rule`** — behavioral constraint

All six ship pre-built. JTBD-first interview pattern: ask about the work, not the schema. Each meta-skill writes a declaration to `.claude/{zone}/` and creates the corresponding folder. Detection clauses fire proactive proposals via `[REVIEW]` inbox.

`/define-skill`, `/define-agent`, `/define-brand` deferred until users explicitly ask. `/define-zone` rejected — five zones is the architecture. Templates and hooks subsumed by other meta-skills or handled at infrastructure level.

### Signals are semantic, not mechanical (Gap #3 resolution)

Signal declarations describe **intent and anchors** — Claude executes the contextual lookup at runtime. No hardcoded windows. No "skip if empty." Claude does graph traversal from today's anchors to whatever's relevant, regardless of how far back the source is.

```yaml
# .claude/signals/daily-plan.md
purpose: |
  Generate today's daily plan. Be contextual and timely.
  Pull whatever's relevant for what's happening today.

anchors:
  zones:
    - today's calendar events
    - today's daily note (if exists; otherwise most recent)
    - gtd/inbox/ items
    - active gtd/projects/ status pages
    - active atlas/initiatives/ status pages
  tools:
    - gws calendar (today + next 3 days)
    - gws gmail (unread)

traversal:
  - For each person on today's calendar: fetch their note + last meeting (no time cap)
  - For each project/initiative referenced today: fetch _status + recent meetings tied to it
  - Surface stale work: projects/initiatives untouched relative to typical cadence

output-format: |
  1. Today's commitments + relevant context for each
  2. Projects/initiatives to advance + where you left off
  3. Stalled items needing attention
  4. Inbox items awaiting triage
```

Production declarations are comprehensive (200+ lines for daily-plan). Bootstrap ships well-designed defaults; `/define-signal` produces a starting shape user iterates on.

**Tools and zones are tried; failures degrade gracefully** — empty zone or uninstalled tool just doesn't contribute that turn.

### Improvability — every declaration is a living document

| Element | Improvement mechanism |
|---|---|
| **Skills** | Per-skill `learnings.md` (existing pattern). Stop hook captures session corrections. |
| **Declarations** (objects, signals, sources, practices, tools, rules) | Inline `## Learnings` section in declaration file. No separate file. Lighter weight. |
| **Cross-declaration patterns** | vault-improvements scans skill learnings.md + declaration `## Learnings` sections + recent corrections in event-log/session-log. Proposes promotion to declaration body, to a rule, or to CLAUDE.md via `[REVIEW]` inbox. |

Three improvement paths for any declaration:
1. **Direct edit** — operator opens declaration and edits
2. **Re-run meta-skill** — `/define-signal` (or future `/update-signal`) walks through changes
3. **Proactive proposals from vault-improvements** — gaps in output detected, edits proposed via `[REVIEW]`

Same pattern as the existing `claude-md-coevolution` rule, applied at the declaration level.

### PostToolUse hook for event-log.md

A `PostToolUse` hook in `.claude/settings.json` fires after every Write/Edit/Bash that touches the vault. Hook script categorizes operations and appends one line to `system/event-log.md` in format `YYYY-MM-DD HH:MM | operation | target | result`. Reading is windowed (7-14 days at session start); older entries roll into `system/event-log-archive/{YYYY-MM}.md` monthly.

### vault-improvements signal mechanism (the self-improvement loop)

Weekly signal that scans the whole vault holistically:
- `system/event-log.md` event stream (recent + archive)
- Unused atlas folders, stale projects, inbox backlog
- Signals not opened, signals firing on empty
- Skill `learnings.md` files
- Declaration `## Learnings` sections
- Captures sitting unprocessed in intake/

Outputs `[REVIEW]` inbox pointers for each holistic recommendation. Operator confirms or dismisses. Vault evolves over time.

### Onboarding failure modes (Gap #9 resolution)

**Phase-by-phase commits.** Each `/onboarding` phase writes results as it goes. If interrupted, partial work is preserved. Resuming picks up at the next incomplete phase.

**Onboarding state file.** `.claude/onboarding-state.md` tracks per-phase progress:

```yaml
---
started: 2026-04-26
phases:
  setup-interview: complete
  demonstrate: incomplete
  graduation: pending
---
```

**Idempotent operations.** Re-running a phase is safe — adding `clients/` when it exists = no-op. No destructive actions during onboarding.

**Skip detection via vault-improvements.** If operator never completes onboarding, vault-improvements detects partial state and surfaces *"Resume `/onboarding`?"* via `[REVIEW]` inbox.

**Sub-commands:**
- `/onboarding` — resumes from incomplete phase, or runs full flow first time
- `/onboarding --status` — shows what's been configured
- `/onboarding --restart` — wipes state with confirmation (vault content untouched)

For adding new engagement containers post-onboarding: run `/define-object` directly. The meta-skill recognizes engagement shape from the JBTD interview answers and scaffolds with the 4-item lighter folder format. No separate `--add-engagement` sub-command.

**Confirmation guards.** Running `/onboarding` after graduation says: *"Onboarding complete. Use --restart to redo everything (vault content is safe)."* No accidental wipes.

### Update path — versioned declarations + 3-way merge (Gap #8 resolution)

**V1 ships these seeds (so V2 can update cleanly later):**
- `version:` field on every shipped artifact (declarations, skills, rules, hook scripts)
- `.claude/defaults/` reference copies of every shipped artifact (V1 baseline)
- `.claude/snapshots/` directory created (empty, ready for pre-update snapshots)
- Bootstrap logs install version to `system/event-log.md`

**V2 ships (deferred — built later):**
- `/workdesk-update` skill that performs 3-way merge:
  1. V1 default (baseline) — `.claude/defaults/{file}`
  2. V2 default (new) — shipped in update
  3. User's current — `.claude/{zone}/{file}`
- For each shipped artifact: auto-apply V1→V2 changes that don't conflict; surface conflicts via `[REVIEW]` inbox (*"keep mine / take V2 / merge"*)
- `/workdesk-update --preview` dry-run mode shows changes without applying
- Auto-snapshot to `.claude/snapshots/{date}-pre-v2/` before any merge
- Migration scripts for schema changes (e.g., new frontmatter field on existing notes)
- After merge, update `.claude/defaults/` to V2 baseline

**What's sacred (never auto-modified):**
- Vault content: atlas/, gtd/, intel/, system/, personal/
- User-created declarations (not based on a shipped default)
- User customizations within shipped declarations (preserved via 3-way merge)

3-way merge is the established pattern (git, npm, package managers). Snapshots provide rollback. Dry-run prevents surprises.

### First-session experience (Gap #7 resolution)

Training videos and live training (provided by Khalil) cover the conceptual layer — 5-zone model, units per zone, atlas vs intel, GTD basics. `/onboarding` focuses on **personalization**, not teaching.

**`/onboarding` skill phases:**
- **Phase A: Setup interview** — concise. *"What kinds of long-running contexts do you have?"*, *"What tools? (Granola, Google Workspace, etc.)"*, *"Daily note practice?"*
- **Phase B: Demonstrate** — generates first daily-plan with light framing
- **Phase C: Graduation** — quick. *"You're set up. Try {X}."*

**Bootstrap → onboarding bridge:**
Bootstrap completes installation, then writes a `[REVIEW]` welcome pointer to `gtd/inbox/`: *"Watch the setup videos at [link], then run `/onboarding`."*

**First-30-days mode (lighter scaffolding):**
- daily-plan output adds tutorial framing for first 14 days
- vault-improvements signal **suppressed for first 14 days** (no patterns yet)
- Empty zone scans return *"this is empty, here's what would go here"*

User flow: install → videos → `/onboarding` → daily use → coached for two weeks → graduates to steady-state.

### Platform scope — Mac-only V1 (Gap #6 resolution)

V1 ships exclusively for macOS. Linux probably works (similar Unix) but isn't officially supported. Windows users see a *"V2 will support Windows"* message if they try to install.

**Mac-only enables:**
- `_workdesk/` symlink to `.claude/` works natively (no Developer Mode required)
- Hook scripts ship as bash in `.claude/scripts/` (no Node.js cross-platform layer)
- Unix paths throughout — no separator conversion
- Single shell-script bootstrap installer

**The vault content architecture is OS-agnostic** — declarations, meta-skills, signals are platform-neutral. Only the install/runtime layer is Mac-specific. V2 adds Windows support by replacing the install/runtime layer; vault content unchanged.

### Migration story — V1 is greenfield only (Gap #10 resolution)

V1 bootstrap **requires an empty vault**. If it detects existing content outside `.obsidian/`, it warns and refuses:

> *"This vault has existing content. WorkDesk OS V1 only supports fresh installs. Options: start a new vault for WorkDesk OS, or wait for the V2 migration assistant. See [training link] for manual migration guidance."*

**Why greenfield-only for V1:**
- Validates the system end-to-end before tackling migration complexity
- Bootstrap stays simple, fewer edge cases
- No risk of clobbering existing user content
- Khalil's own migration is in the same boat — confirmed will be a separate session

**Bootstrap detection logic:**
```
if vault has any content outside .obsidian/:
  warn + refuse install
  point to manual-migration documentation
else:
  proceed with fresh install
```

**V2 ships migration assistant (deferred):**
- `/migrate` skill that detects existing content, maps to WorkDesk OS structure
- Walks operator through proposed moves (per file, per folder)
- Executes with full backup snapshots before any change
- Not in V1.

## Critical Files To Be Created

### Bootstrap installer
- `bootstrap.sh` — Mac shell script: verifies macOS, detects empty vault (refuses if not), creates folder structure, installs hooks, sets `_workdesk/` symlink, writes welcome `[REVIEW]` to `gtd/inbox/`, logs install version to `event-log.md`

### Five zone folders (created by bootstrap, mostly empty on day 1)
- `personal/daily/` (the one universal practice)
- `atlas/meetings/`, `atlas/decisions/`, `atlas/people/`, `atlas/initiatives/`
- `gtd/inbox/`, `gtd/actions/next/`, `gtd/actions/waiting/`, `gtd/projects/`, `gtd/someday/{actions,projects}/`, `gtd/archive/{projects,actions}/`
- `intel/briefings/daily/`, `intel/vault-improvements/`, `intel/research/`, `intel/observations/`
- `system/intake/`, `system/transcripts/`, `system/session-log/`, `system/event-log.md`, `system/event-log-archive/`

### `.claude/` infrastructure
- `.claude/settings.json` — declares PostToolUse hook for event-log + Stop hook for learnings (existing pattern)
- `.claude/scripts/post-tool-use-log.sh` — bash hook script (Mac-only V1)
- `.claude/scripts/monthly-rotate-event-log.sh` — rolls event-log.md to event-log-archive monthly
- `.claude/scripts/bootstrap-vault.sh` — vault-content scaffolding helper
- `.claude/skills/` — 11 core skills:
  - `/onboarding` (with phases A–D and `--status` / `--restart` sub-commands)
  - `/daily-ops`
  - `/extract`
  - `/obsidian-markdown`
  - `/pobo`
  - `/create-spec`
  - `/codex-rescue`
  - `/define-object`
  - `/define-signal`
  - `/define-source`
  - `/define-practice`
  - `/define-tool`
  - `/define-rule`
- `.claude/rules/` — universal rules (no-fabrication, source-documentation, matching, double-entry-knowledge, writing-style, claude-md-coevolution, per-project-accounting) + universal tool refs (obsidian-cli, qmd, defuddle)
- `.claude/objects/` — declarations for action, project, inbox-item, meeting, decision, person, initiative
- `.claude/signals/` — declarations for daily-plan, vault-improvements (each with `## Learnings` section seeded empty + `version: 1.0`)
- `.claude/sources/` — declarations for transcript, session-log, intake
- `.claude/practices/` — declaration for daily
- `.claude/defaults/` — reference copies of every shipped artifact (V1 baseline for future V2 merge)
- `.claude/snapshots/` — empty, ready for pre-update snapshots
- `.claude/templates/` — format scaffolds for meta-skills (e.g., starter object format, starter signal format)
- `.claude/onboarding-state.md` — tracks per-phase onboarding completion

### Existing files to reuse / adapt
- `.claude/hooks/stop-learnings.sh` — already exists, reused for `## Learnings` section + skill learnings.md scanning
- `.claude/rules/per-project-accounting.md` — already exists at 8-item structure (recently updated 2026-04-23)
- `.claude/skills/{obsidian-cli,qmd,defuddle,pobo,daily-ops,extract,obsidian-markdown}/` — already exist, reused
- `.claude/skills/onboard-client/` — adapted into the broader `/onboarding` skill or kept as a sub-flow
- `.claude/agents/orchestrator.md` — kept; routes between skills
- `system/log.md` already exists at 91 lines — V1 renames to `event-log.md`; existing entries preserved

## Verification

End-to-end smoke test on a fresh Mac (or fresh test vault):

### 1. Bootstrap test
- Point bootstrap at empty test vault → installs cleanly
- Point bootstrap at vault with existing content → refuses gracefully with migration message
- Verify `_workdesk/` symlink renders in Obsidian and edits propagate to `.claude/`
- Verify `personal/` is hard-locked (try to write via Claude Code → blocked)

### 2. Onboarding test
- First Claude Code session → finds welcome `[REVIEW]` in `gtd/inbox/`
- Run `/onboarding` → completes Phase A (engagement containers), Phase B (demonstrate), Phase C (first daily-plan), Phase D (graduation)
- Verify `.claude/onboarding-state.md` updates per-phase
- Interrupt mid-Phase B → re-run → resumes at Phase B (idempotent)
- Run `/onboarding --status` → reports correctly
- Run `/onboarding --restart` → wipes state with confirmation prompt

### 3. Object lifecycle test
- Drop a meeting transcript in `system/transcripts/` → processes into `atlas/meetings/{date}-{slug}.md` + creates/updates `atlas/people/`, `atlas/decisions/` as appropriate
- Verify source frontmatter on transcript flips `processed: true` with `processed-into:` backlinks
- Verify `event-log.md` records each operation as one line
- Verify low-confidence creations drop `[REVIEW]` pointers in `gtd/inbox/`

### 4. Signal test
- Run `/daily-ops` → invokes daily-plan signal → produces `intel/briefings/daily/{date}-daily-plan.md` with first-30-days framing
- Verify graceful degradation on empty zones (sparse vault still produces useful plan)
- Verify graceful skip on uninstalled tools (e.g., gws not installed → calendar source skipped)

### 5. GTD lifecycle test
- POBO produces project in `gtd/projects/{slug}/` with full 8-item structure
- Promote next physical action → creates `gtd/actions/next/{slug}.md` with `parent:` link to project
- Move action to `gtd/actions/waiting/` → status updates work
- Move action to `gtd/archive/actions/{year-month}/` → archive structure works

### 6. Meta-skill test
- Run `/define-object` → JBTD interview produces a working `.claude/objects/{type}/` declaration + scaffolded folder
- Run `/define-signal` → produces a working `.claude/signals/{type}/` declaration with anchors + traversal + output-format + `## Learnings` section seeded empty
- Run `/define-tool` → adds tool reference rule + permission updates

### 7. Hook + log test
- Verify PostToolUse hook fires on every Write/Edit/Bash → appends one line to `event-log.md`
- Verify monthly rotation hook works (force-trigger rotation, see `event-log-archive/{YYYY-MM}.md` created)
- Verify session start reads only last 7-14 days of `event-log.md`

### 8. Codex review (this plan)
- Run `/ultrareview` against this plan file before any implementation begins
- Codex returns confidence rating + adversarial findings
- Address findings in plan before V1 build kicks off

## Source

POBO planning sessions 2026-04-17 → 2026-04-26 with Khalil. Codex adversarial review 2026-04-19. Walkthrough sessions 2026-04-23 → 2026-04-26.
