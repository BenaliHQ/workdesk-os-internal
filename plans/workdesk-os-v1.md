# WorkDesk OS — Five-Zone Vault Plan

## Context

This plan crystallizes the five-zone WorkDesk OS architecture so it can be reviewed by Ultra Review / Codex before implementation begins. The vault has lived in a four-zone model (personal, atlas, intel, system) since vault-architecture; testing with Jenny Meier and the Codex POBO review revealed that GTD-style action management was scattered across project notes, meeting notes, and inline checkboxes with no canonical home. Splitting GTD into its own zone resolves the tension and gives each zone a single unit type to manage.

The output of this plan is a working WorkDesk OS bootstrap that any user can install on a fresh machine, scaffold a starter vault, and use immediately — with Claude proactively extending the vault over time via four meta-skills and a self-improvement loop.

## Five-Zone Model

Each zone manages one unit type. Each zone has one job.

**What "locked" means in this plan.** When a zone is marked *locked*, it means the architecture decision is committed for V1: the unit, the job, the agent-write policy, and the top-level shape are not up for revision in this build. Operators still extend each zone via meta-skills (`/define-object`, `/define-signal`, etc.). "Locked" is a design commitment to stop re-litigating zone boundaries during implementation, **not** a permanent law — V2+ may revisit if real usage exposes a sixth zone need. `/define-zone` is rejected for V1 specifically; it is not philosophically forbidden forever.

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

**Hard-locked read-only.** A `PreToolUse` hook in `_workdesk/settings.json` blocks Write, Edit, and NotebookEdit operations whose path resolves under `personal/`. Enforcement is at the harness level — not just instruction. Reads (Read, Glob, Grep) are allowed.

**Practice declaration** at `_workdesk/practices/{name}.md`: identity, cadence, template, read policy, detection clause.

**Pre-built declarations shipped with bootstrap:**
- `daily` — daily note practice (cadence: daily; template: minimal; read-policy: morning daily-plan reads today's note)

**`/define-practice` meta-skill** scaffolds new practices through a JTBD-first interview. Onboarding may propose common ones (journal, reading) but never auto-creates them.

**Migration:** out of scope for this bootstrap. The existing `personal/journal/`, `personal/notes/`, `personal/books/`, `personal/identity/` content stays where it is in Khalil's current vault. The fresh bootstrap delivers only the read-only lock + daily practice. Migration happens in a separate session after operator validates the bootstrap.

### Zone 2: Atlas — locked

**Unit:** object. **Job:** object management — single-source identified evidence. **Agent writes?** Yes.

**Source rule:** atlas notes carry claim-level source attribution per the source-documentation rule. A note may aggregate multiple sources over time (a person note accumulates from many meetings; an engagement status updates session after session) — but every claim cites where it came from. Atlas answers "did this happen / does this exist?" Intel asks "what does this mean for me?" Multi-source is fine in both zones; the split is purpose.

**Folder per object type. Universal-five ship; engagement containers are optional overlays; everything else is emergent:**

```
atlas/
  meetings/        # universal — ships pre-built — single notes
  decisions/       # universal — ships pre-built — single notes
  people/          # universal — ships pre-built — single notes
  initiatives/     # universal — ships pre-built — folders, 8-item structure each
  areas/           # universal — ships pre-built — folders, 4-item structure each
  {engagement-containers}/  # OPTIONAL — added during onboarding when role demands
  {emergent-types}/         # added later via vault-improvements signal proposals
```

**Why `areas/` ships universal.** `areas/` is the durable container for ongoing responsibility that isn't relationship-shaped:
- consultant: client-independent admin — finance, pipeline, health
- founder: company-wide functions — hiring, runway, legal, ops
- employee: role, career, team health, manager relationship
- researcher: methods, teaching, reading, lab ops
- creative: studio, audience, publishing
- parent: household, family, school, health

The Atlas/engagement test:
- *Named, ongoing, relationship-shaped* (a client, business, team, lab, etc.) → engagement container
- *Enduring but not relationship-shaped* → `areas/`
- *Bounded outcome with multi-session work* → `initiatives/`

**Areas folder shape — 4-item (same as engagement folders):**

```
atlas/areas/{slug}/
  _brief.md       # what this area covers, why it's a standing concern
  _status.md      # current state, recent activity, open threads
  notes/          # ongoing observations
  _archive/       # retired material
```

Areas are not initiatives. Initiatives have a bounded outcome and end; areas are permanent until retired. Recurring maintenance work for an area lives in `gtd/recurring/`, not as initiatives.

**Engagement containers — optional, surfaced in onboarding.** Onboarding's role-map phase asks *"What kinds of named, ongoing, relationship-shaped contexts do you work in?"* Operator picks zero or more. **A user can be consultant + founder + parent concurrently** (see `_workdesk/operator-profile.md`); engagement containers and `areas/` coexist freely.

| Role-map prompt | Resulting container | 4-item folder per instance | Typical instances |
|---|---|---|---|
| *"Consultant?"* | `clients/` | `_brief.md`, `_status.md`, `notes/`, `_archive/` | acme, dudley, yc-batch |
| *"Founder/owner?"* | `businesses/` | same | benali, growthkits |
| *"Employee at an organization?"* | `teams/` or `departments/` | same | platform-team, design-org |
| *"Researcher/academic?"* | `collaborations/` or `labs/` | same | lab-x, nih-grant-2026 |
| *"Creative work?"* | `disciplines/` (or user-named) | same | photography, novel, songwriting |

If the operator picks none, only the universal-five ship — and that's a complete, usable vault. Engagement containers can be added later via `/define-object`.

**Persona caveat — work-centric framing.** Some language in this plan ("operator", POBO, "session") tilts knowledge-work. Personal-life users (parent managing household, hobbyist, retiree) get the same architecture and `areas/` is the universal container that makes their day-1 fit clean. Daily-plan tonality may still feel work-shaped initially; first-30-days mode softens this, and personal-mode tonality ships as a deferred V1.x improvement.

**Emergent objects (NOT in onboarding — proposed by vault-improvements signal over time):**

| Object type | Trigger for proposal |
|---|---|
| `companies/` | Repeated org references across meetings/transcripts |
| `content/` | Content-production patterns or explicit "draft" / "publish" notes |
| Anything else (deals, listings, book-notes, grants, etc.) | Pattern detection via vault-improvements |

**Two folder shapes for atlas objects:**
1. **Atomic objects** (meetings, decisions, people, companies-when-emergent) → single notes per instance
2. **Container objects with ongoing context** (initiatives, areas, engagements like clients/businesses/teams/labs/etc., future deals/listings) → folders with their own minimum structure

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

**Object declaration** at `_workdesk/objects/{type}.md` carries:
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

**The action vs project vs initiative vs recurring test (universal):**
- *Repeating commitment with cadence* → **recurring** (`gtd/recurring/`)
- *Single-session execution by Claude or operator* → **action** (`gtd/actions/next/`), can link to engagement or area directly
- *Multi-session operator attention required, no engagement* → **project** (`gtd/projects/`)
- *Multi-session engagement-tied or bounded outcome* → **initiative** (`atlas/initiatives/`) — typically with `engagement:` or `area:` parent

Tasks do not exist as a separate unit. Multi-session = project or initiative. Single-session = action. Repeating = recurring. Initiatives are reserved for **bounded outcomes** (a paper revision, a grant submission, a launch); standing responsibility lives in `areas/` with recurring items underneath. This prevents initiative sprawl (especially for parents and employees, where most work is recurring maintenance).

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
  recurring/          # repeating commitments — cadence-bound or checklist-shaped
    schedules/        # cadence-bound (weekly client prep, payroll, school logistics)
    checklists/       # repeatable procedures (publish workflow, expense close)
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

**`parent:` is polymorphic.** One field, points to whatever owns the action. Claude dereferences the link to figure out the parent's type. Allowed parents:
- `parent: "[[gtd/projects/workdesk-os]]"` — owned by a personal-focus project
- `parent: "[[atlas/initiatives/dudley-msa-review]]"` — owned by an engagement-tied initiative
- `parent: "[[atlas/areas/household]]"` — tied to a standing area of responsibility
- `parent: "[[atlas/clients/dudley]]"` — tied directly to an engagement, no project/initiative wrapping
- `parent: "[[gtd/recurring/schedules/weekly-payroll]]"` — promoted from a due recurring item
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

**Recurring format — two shapes:**

Schedules (cadence-bound):

```yaml
---
type: recurring
shape: schedule
status: active                                          # active | paused | retired
cadence: weekly                                         # daily | weekly | monthly | quarterly | yearly | custom
next_due: 2026-05-01
parent: "[[atlas/areas/household]]"                     # area, engagement, project, or empty
source: ""
created: 2026-04-26
---
Weekly school logistics review — confirm pickups, lunches, after-school.
```

Checklists (repeatable procedures, not cadence-bound):

```yaml
---
type: recurring
shape: checklist
status: active
parent: "[[atlas/areas/publishing]]"
source: ""
created: 2026-04-26
---
# Publish workflow

- [ ] Final read-through
- [ ] Run spellcheck
- [ ] Generate share image
- [ ] Schedule social posts
- [ ] Cross-post to mailing list
```

**Recurring lifecycle:**
- Recurring items live in `gtd/recurring/`, **not** in `actions/next/` permanently
- **weekly-review** signal scans `gtd/recurring/schedules/` for items where `status: active` AND `next_due <= today + 7 days`
- **daily-plan** signal includes due items in its top section, filtered to `status: active`
- **vault-improvements** "recurring items overdue" check filters to `status: active`
- **Promotion to action** is operator-confirmed (or auto when `cadence: daily` and operator opts in): creates a transient action in `gtd/actions/next/` with `parent: "[[gtd/recurring/schedules/{slug}]]"`
- **Completion** of the promoted action triggers `next_due` roll-forward by the cadence (weekly → +7d, monthly → +1mo); event logged
- **Pause** (`status: paused`) — drops out of all scans; `next_due` does not roll. Resume by setting `status: active`.
- **Retire** (`status: retired`) — terminal state; drops out of all scans permanently. File stays in place for historical reference; not moved to archive (no `gtd/archive/recurring/` folder in V1).
- Checklists are not promoted automatically; operator runs them via `/checklist {slug}` which materializes the steps as actions

This keeps GTD's clarity (actions are still the unit of doing) while making real-life maintenance work first-class.

**Inbox format:**

```yaml
---
type: inbox-item
prefix: REVIEW                                         # REVIEW | ACTION | QUESTION | AWARENESS
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

**Inbox flood guard.** The risk: every detection across atlas/intel/sources drops `[REVIEW]` items and operator drowns. Mitigations shipped in V1:
- **Per-session cap** — Claude drops at most 7 new `[REVIEW]` items per session. Past the cap, additional detections are batched into a single rolled-up `[REVIEW]` ("12 more potential people-note creations from this transcript — review batch?").
- **Confidence threshold** — proposals require ≥0.7 self-rated confidence to drop a `[REVIEW]`. Below that bar, Claude either asks inline ("Want me to create a person note for {name}?") or stays silent. The 0.7 threshold is a starting heuristic, tunable per-declaration via `proposal-threshold:` in the declaration frontmatter.
- **Inbox backlog signal** — if `gtd/inbox/` exceeds 20 unresolved items, daily-plan surfaces a triage prompt as its top item.
- **Auto-expiry by prefix** — `[AWARENESS]` after 7 days; `[QUESTION]` after 14 days; `[REVIEW]` and `[ACTION]` never expire (operator clears).

**POBO → project flow:** POBO produces `_brief.md`, `_status.md`, `plan.md`. The plan holds intended phase steps as text. **Only the current physical next action gets promoted** to `gtd/actions/next/` as an action object. Future steps stay in `plan.md` until promotion. Promotion is always agent-proposed, operator-confirmed.

**Lite-POBO path.** Full POBO is a multi-question ritual — appropriate for high-stakes initiatives, friction for low-stakes work. When operator wants to create a project but full POBO feels heavy, `/pobo --lite` produces a 3-field minimum: outcome (one sentence), first next action (one line), end-state signal (how you'll know it's done). `_brief.md` and `_status.md` are stubbed; `plan.md` is empty until operator returns. Folder still has the 8-item structure. vault-improvements proposes upgrading to full POBO if the lite project gains 3+ actions or runs longer than 2 weeks.

**Default at action-creation time.** When operator creates work and the project-vs-action test is ambiguous ("am I going to be engaged across multiple sessions?"), default to **action**. Promote to project later via `/promote-to-project {action-slug}` if it grows. This is the lower-cost mistake — orphaned projects (over-promotion) sit empty and shame the operator; over-flat actions just need promotion when they earn it.

**Pre-built declarations shipped with bootstrap:**
- `_workdesk/objects/action.md` — action format + lifecycle
- `_workdesk/objects/project.md` — project format + scan locations (`gtd/projects/`, `atlas/initiatives/`)
- `_workdesk/objects/recurring.md` — recurring format + lifecycle (schedules + checklists)
- `_workdesk/objects/inbox-item.md` — inbox format + prefix rules

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
      2026-04-20-weekly-review.md
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

**Signal declaration** at `_workdesk/signals/{type}.md`:
- **Identity** — name, folder location, file-naming pattern
- **Sources** — what it gathers from (calendar, email, atlas, `system/events/`, etc.)
- **Format** — frontmatter + body sections Claude follows
- **Schedule** — on-demand, daily, weekly, event-triggered
- **Output** — where it lands; whether it drops a `[REVIEW]` inbox pointer
- **Detection** — when Claude proposes generating one ad-hoc

**Pre-built signals shipped with bootstrap (three only):**

**1. `daily-plan`** (`_workdesk/signals/daily-plan.md`)
- Sources: today's calendar, unread email, today's daily note, yesterday's meeting transcripts, recent events, active gtd/projects/ statuses, active atlas/initiatives/ statuses, due recurring items
- Schedule: daily (first-session-after-midnight trigger or on-demand via `/daily-ops`)
- Output: `intel/briefings/daily/{date}-daily-plan.md`

**2. `weekly-review`** (`_workdesk/signals/weekly-review.md`) — **mandatory in V1**
- Sources: open actions, projects, initiatives, areas, recurring items due in next 7 days, inbox backlog, stale contexts (not touched > area-typical cadence), processed transcripts, last week's daily notes
- Schedule: weekly (first session of the week or on-demand via `/weekly-review`)
- Output: `intel/briefings/weekly/{date}-weekly-review.md` + `[REVIEW]` inbox pointers for proposed closures, promotions, and cleanup
- The bridge between onboarding and stable use. Active from week 1. More important for early retention than vault-improvements.

**3. `vault-improvements`** (`_workdesk/signals/vault-improvements.md`)
- Sources: `system/events/` monthly files, unused atlas folders, stale projects, inbox backlog, signals not opened, captures sitting unprocessed, broken wikilinks, missing required frontmatter, oversized media
- Schedule: weekly — but **suppressed for first 14 days** (no patterns yet)
- Output: `intel/vault-improvements/{date}-vault-improvements.md` + drops `[REVIEW]` inbox pointers for each recommendation
- Holistic scan across atlas, gtd, intel, personal, system — whole-vault, not per-zone
- The self-improvement loop. Ships pre-built but stays out of the way until weekly-review and daily-plan have stabilized.

**Daily-plan must succeed across three data conditions.** Sparse-data and offline-manual users get a useful plan, not a hollow summary. Fallback chain (any sources present in any layer produce content; layers are tried in order, layers that produce nothing are silently skipped):

1. Calendar commitments (today + tomorrow)
2. `gtd/actions/next/` (sorted by `parent:` recency)
3. Due recurring items from `gtd/recurring/schedules/`
4. Active project and initiative `_status.md` summaries
5. Today's daily note (if exists)
6. Unread inbox items (with backlog warning if > 20)
7. Stale contexts needing attention (areas/initiatives untouched relative to typical cadence)

If layers 1–7 all return empty (true cold-start), daily-plan produces a **setup-oriented plan**: *"Nothing scheduled and nothing in next-actions. First steps to seed the vault: …"* — not a hollow report.

**`/define-signal` meta-skill** scaffolds new signals via JTBD-first interview:
- *"What do you want Claude to keep an eye on for you?"*
- *"What sources should it look at?"*
- *"How often should it run?"*
- *"What should it produce?"*

Meta-skill writes `_workdesk/signals/{name}.md`, creates `intel/{name}/` folder, optionally schedules.

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
  session-log/         # per-session narratives, written by /extract
    2026-04-26-{slug}.md
  events/              # append-only monthly event files (replaces single rolling event-log.md)
    2026-04.md
    2026-05.md
  media/               # binaries — see "Binaries and media" below
    2026-04/
```

**Why monthly event files instead of a single rolling log:** simpler implementation, no rotation race condition, append-only semantics are trivial, bounded reads (last 1-2 monthly files cover a typical session window). The rotation hook is gone; the file path itself encodes the rotation.

**Raw sources persist after processing.** Transcripts and captures stay in their folder — never deleted. Frontmatter flips `processed: true` and `processed-into: ["[[...]]"]` lists the atlas/intel notes that came from it. Per-source-type rule: `move-after-processing: false | "_archive/{year-month}/"`. Default = false (keep in main folder).

**Binaries and media** — images, audio, video, screenshots, PDFs live in `system/media/` co-located with the source that produced them:

```
system/media/
  {YYYY-MM}/
    {date}-{slug}-{kind}.{ext}    # 2026-04-26-dudley-weekly-recording.m4a
```

Atlas/intel notes embed media via standard Obsidian links (`![[system/media/2026-04/2026-04-26-dudley-recording.m4a]]`). Source declaration's `processing rule` may strip large media from atlas notes (link only, don't embed) to keep notes light. No size cap enforced by V1; vault-improvements surfaces oversized media monthly.

**Universal source frontmatter:**

```yaml
---
type: source
source-kind: transcript | session-log | intake | other
date: 2026-04-26
processed: false                # flips to true after atlas/intel extraction
processed-into: ["[[...]]"]    # backlinks to notes produced from this source
---
```

**Source declaration** at `_workdesk/sources/{type}.md`:
- **Identity** — name, folder location, file pattern
- **Format** — what raw input looks like
- **Processing rule** — how it becomes atlas/intel/gtd notes
- **Retention** — keep forever, archive after N days, delete after N days

**Pre-built source declarations (universal — V1 ships exactly three):**
- `transcript` — Granola/Google Meet/manual transcripts. Processing rule: extract meeting → `atlas/meetings/` + decisions → `atlas/decisions/` + people updates
- `session-log` — claude-log entries. Written by `/extract` at session end. Format: summary section (wikilink-able) + full conversation appended verbatim
- `intake` — generic raw drops. Processing rule: triage to atlas/intel/gtd

**Conditional source declarations (V1.x or later — explicitly NOT in V1):**
- `bookmark` — only if user installs Defuddle/Keep.md (proposed by vault-improvements when first article URL appears in intake/)
- `email-export`, `screenshot`, `pdf`, `ocr-scan` — deferred. Narrow source scope avoids brittle integrations during V1 launch.

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

**Event logging mechanism (hook-driven, semantic events only):**

A `PostToolUse` hook in `_workdesk/settings.json` fires after Write/Edit/Bash, but it does **not** log every operation. The hook categorizes and only logs **high-value semantic events** to keep the log signal-rich. V1 logs exactly these event classes:

1. `bootstrap-install-completed`
2. `onboarding-phase-completed`
3. `source-processed` (transcript / intake → atlas-or-gtd notes)
4. `object-created` (atlas object: meeting, decision, person, area, initiative, engagement instance)
5. `project-created`
6. `initiative-created`
7. `action-promoted` (recurring → next, or project-plan → next)
8. `action-completed` (next → archive, or recurring `next_due` rolled forward)
9. `signal-generated` (daily-plan, weekly-review, vault-improvements)
10. `declaration-changed` (object/signal/source/practice/tool/rule edited)
11. `object-archived` (project, initiative, engagement, or area moved to archive; inbound-link rewrites recorded in the same event)

The parenthetical scopes in classes 4 and 9 are illustrative, not exhaustive. Any atlas object type declared via `/define-object` produces `object-created` events; any signal type declared via `/define-signal` produces `signal-generated` events. The hook categorizes by zone-appropriate write, not by the named subtype.

Write line format: `YYYY-MM-DD HH:MM | event-class | target | result-or-context`. Append-only into the current month's file: `system/events/{YYYY-MM}.md`. No rotation hook needed — month rollover happens because the path includes the month.

**Reading is windowed.** Session start reads the current month's file plus the previous month's file (covers any 7-30 day window). Older months stay readable on demand.

**session-log/ vs events/:**
- `session-log/` = per-session narrative ("what we discussed and decided this session"). Written by `/extract`.
- `events/` = per-event semantic stream. Written by hooks; one line per high-value event.
- A single session typically produces 1 entry in `session-log/` and 5-30 entries across `events/`.

**Reference / scripts / templates live in `_workdesk/`:**

These are infrastructure, not sources. `system/` is now purely sources + activity logs.

```
_workdesk/
  scripts/           # hook scripts, automation
  templates/         # reusable scaffolds for meta-skills
  hooks/             # hook definitions referenced from settings.json
```

SOPs and workflows are NOT shipped in V1. Future `/define-sop` or `/define-workflow` meta-skills if users ask.

**Control-plane visibility — `_workdesk/` is the source of truth:**

Bootstrap creates `_workdesk/` as a real directory at vault root (underscore sorts to top in Obsidian; visible without "Show Hidden Files" plugins). All declarations, hooks, scripts, settings live in `_workdesk/`.

For tools that expect `.claude/` (Claude Code itself, gstack, other Anthropic tooling), bootstrap creates a **`.claude` symlink** pointing at `_workdesk/`:

```
_workdesk/      # real directory, visible, primary source of truth
.claude         # symlink → _workdesk/  (compatibility alias)
```

Editing `_workdesk/{file}` IS editing `.claude/{file}` — same underlying file. Operators inspect, edit, and reason about `_workdesk/`; Claude Code reads through `.claude/` natively. **If end-to-end testing reveals Claude Code requires a physical `.claude/` directory rather than tolerating a symlink**, bootstrap inverts the relationship (`.claude/` real, `_workdesk/` symlink to it). One model is picked and tested before V1 ships; the operator-facing semantics are identical either way.

**Sync-layer caveats (Mac):**
- **iCloud Drive** — symlinks survive but the target may be evicted to "Optimize Mac Storage." If Claude Code reports missing files, force-download via `brctl download _workdesk/`. Recommended: keep the vault out of iCloud, or pin `_workdesk/` to "Always Keep on This Device."
- **Obsidian Sync** — does not sync symlinks. The `.claude` alias exists only on the device that ran bootstrap; other devices see `_workdesk/` (the real directory) directly. Vault content (atlas/, gtd/, intel/, system/, personal/) syncs normally.
- **Dropbox / Google Drive** — vary by provider; if either path shows as a 0-byte file in Obsidian on another device, the provider is following the link. Resolution: turn off "follow symlinks" in the sync client, or keep Claude Code use confined to one device.

### Operator profile — first-class artifact for mixed personas

V1 ships `_workdesk/operator-profile.md` as a required control-plane file. This is what makes mixed-persona users workable — a person can be consultant + founder + parent concurrently, and the profile captures all three.

**Format:**

```yaml
---
name: Khalil Benalioulhaj
role-mix:
  - consultant         # primary or secondary; order matters for tonality
  - founder
  - parent
primary-contexts:
  engagements:         # active engagement-container instances
    - "[[atlas/clients/dudley]]"
    - "[[atlas/businesses/benali]]"
  areas:               # active area instances
    - "[[atlas/areas/family]]"
    - "[[atlas/areas/health]]"
enabled-tools:         # detected during onboarding; influences signal sources
  - granola
  - gws-calendar
  - gws-gmail
  - qmd
preferred-naming:
  engagements: "kebab-case"
  people: "first-last"
daily-planning-style: "morning"   # morning | end-of-day | on-demand
week-start: "monday"              # monday | sunday
first-30-days-mode: active        # active | graduated
created: 2026-04-26
last_updated: 2026-04-26
---
```

**How it's used:**
- `daily-plan` reads `role-mix` to decide tonality and section ordering (consultant-first plans look different from parent-first plans)
- `weekly-review` reads `primary-contexts` to scope the scan
- `enabled-tools` gates which sources signals try (no GWS → no calendar source)
- `/define-object` and `/onboarding` propose containers consistent with `role-mix`
- `vault-improvements` proposes profile updates when usage drifts (e.g., "you've stopped touching `atlas/clients/`; mark consultant role retired?")

**Profile is editable.** Operators update it directly; `/onboarding --update-profile` walks them through changes interactively.

## Cross-Cutting Concerns

### Six meta-skills (final set)

1. **`/define-object`** — atlas content types
2. **`/define-signal`** — intel signal types
3. **`/define-source`** — system source types
4. **`/define-practice`** — personal practice types
5. **`/define-tool`** — Claude capability/integration (CLI, API, MCP)
6. **`/define-rule`** — behavioral constraint

All six ship pre-built. JTBD-first interview pattern: ask about the work, not the schema. Each meta-skill writes a declaration to `_workdesk/{zone}/` and creates the corresponding folder. Detection clauses fire proactive proposals via `[REVIEW]` inbox.

`/define-skill`, `/define-agent`, `/define-brand` deferred until users explicitly ask. `/define-zone` rejected — five zones is the architecture. Templates and hooks subsumed by other meta-skills or handled at infrastructure level.

### Signals are semantic, not mechanical (Gap #3 resolution)

Signal declarations describe **intent and anchors** — Claude executes the contextual lookup at runtime. No hardcoded windows. No "skip if empty." Claude does graph traversal from today's anchors to whatever's relevant, regardless of how far back the source is.

```yaml
# _workdesk/signals/daily-plan.md
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
| **Cross-declaration patterns** | vault-improvements scans skill learnings.md + declaration `## Learnings` sections + recent corrections in `system/events/` and `system/session-log/`. Proposes promotion to declaration body, to a rule, or to CLAUDE.md via `[REVIEW]` inbox. |

Three improvement paths for any declaration:
1. **Direct edit** — operator opens declaration and edits
2. **Re-run meta-skill** — `/define-signal` (or future `/update-signal`) walks through changes
3. **Proactive proposals from vault-improvements** — gaps in output detected, edits proposed via `[REVIEW]`

Same pattern as the existing `claude-md-coevolution` rule, applied at the declaration level.

### PostToolUse hook for system/events/

A `PostToolUse` hook in `_workdesk/settings.json` fires after Write/Edit/Bash that touches the vault. Hook script categorizes the operation against the 10 semantic event classes (see Zone 5 system events spec) and appends one line to `system/events/{current-YYYY-MM}.md`. Operations that don't match a semantic class are dropped silently — that's the point of narrowing.

**Concurrency:** parallel tool calls can fire the hook simultaneously. Hook script acquires an advisory lock with `shlock(1)` (BSD-native, no brew dependency) on a sibling `.events.lock` before appending. On lock-acquisition failure, retry up to 3× with 50ms backoff, then drop the entry and emit a stderr warning (operations succeed; only the log entry is lost). Dropped entries are acceptable — the log is observability, not a database.

**Hook overhead budget:** total hook latency must stay under 50ms p95 to avoid perceptible slowdown in Claude Code. Bootstrap installs a benchmark script at `_workdesk/scripts/bench-hooks.sh` that operator can run anytime to confirm the hook stays within budget.

**Why fewer events = better.** All-event logging creates noise, performance drag, and failure points. V1 logs only what review signals actually need; vault-improvements scans `events/` for patterns; daily-plan and weekly-review consume the high-value classes directly.

### Signal scheduling — how daily/weekly cadence actually fires

Signal declarations carry `schedule: daily | weekly | on-demand | triggered`, but Claude Code itself doesn't have a scheduler. V1 mechanisms by cadence:

| Cadence | Mechanism |
|---|---|
| `on-demand` | Operator runs `/daily-ops`, `/weekly-review`, etc. |
| `daily` | First Claude Code session after midnight checks signal "last-fired" timestamp; if stale, proposes running it. No background daemon. |
| `weekly` | Same first-session check; weekly-review is the canonical weekly (vault-improvements suppressed first 14 days, then weekly). |
| `triggered` | Detection clauses fire inside meta-skills (e.g., new transcript in `system/transcripts/` triggers processing). |

**Trade-off:** signals don't fire if operator never opens Claude Code. This is acceptable for V1 — daily-plan and weekly-review are only useful inside a Claude Code session. A future V1.x cron-based runner ships if this assumption breaks.

### vault-improvements signal mechanism (the self-improvement loop)

Weekly signal — **suppressed for first 14 days** so weekly-review stabilizes first — that scans the whole vault holistically:
- `system/events/` monthly files (recent + prior month)
- Unused atlas folders, stale projects, inbox backlog
- Signals not opened, signals firing on empty
- Skill `learnings.md` files
- Declaration `## Learnings` sections
- Captures sitting unprocessed in intake/
- **Broken wikilinks** — Obsidian-style `[[...]]` references whose target file no longer exists (typically caused by archive moves, renames, or manual deletes)
- **Missing required frontmatter** — atlas notes without `source:`, signals without `sources:`, etc.
- **Oversized media** in `system/media/` (>50MB single file)
- **Recurring items overdue** (`status: active` items where `next_due` is past today by > one cadence interval)

Outputs `[REVIEW]` inbox pointers for each holistic recommendation. Operator confirms or dismisses. Vault evolves over time.

**Archive moves and link integrity.** When a project, initiative, or engagement is archived, its path changes (e.g., `gtd/projects/foo/` → `gtd/archive/projects/2026/foo/`). Polymorphic `parent:` links from actions, plus any `[[wikilinks]]` from atlas/intel notes, will break. Resolution:
1. **Agent-driven moves rewrite links** — when Claude performs the archive move, it scans for inbound references and updates them in the same operation, logging the rewrite as an `object-archived` event to `system/events/`.
2. **Operator-driven moves (drag in Obsidian) get caught by vault-improvements** — broken-link scan surfaces them as `[REVIEW]` items the next time the signal runs.
3. **Renames follow the same pattern** — agent rewrites; manual rename → vault-improvements catches.

### Onboarding — six phases, self-sufficient

V1 onboarding stands on its own. Training videos (when available) explain the conceptual layer faster, but **onboarding does not require them**. Hard limit: at most one major conceptual choice per phase.

**Six phases:**

1. **Environment check** — verify install health, confirm `_workdesk/` resolves, confirm `personal/` lock is active, report what's already in place. Read-only — no questions yet.
2. **Role map** — capture mixed-persona profile. *"Which of these describe how you spend your time? (pick one or more)"* — produces `_workdesk/operator-profile.md` with `role-mix`. No JBTD interview yet; just the role list.
3. **Context setup** — based on role mix: create `atlas/areas/` instances first (always), then optional engagement containers. *"You picked consultant + parent. Want to scaffold `atlas/clients/` and `atlas/areas/family`, `atlas/areas/household` now?"* Operator confirms or skips per item.
4. **Tool setup** — detect optional connectors (Granola, GWS, qmd, Defuddle). For each: present, run a smoke test, record result in `enabled-tools`. **Missing tools never block** — they just degrade the signals that depend on them.
5. **First daily plan** — generate `intel/briefings/daily/{today}-daily-plan.md` from whatever data exists. Sparse-vault path: surface the 7-step fallback chain output. Cold-start path: produce the setup-oriented plan. Either way, operator sees something useful.
6. **Graduation** — quick. Two-action close: *"Use the daily-plan tomorrow morning. Run `/weekly-review` at end of week."* That's it. Advanced modeling moves to normal use via `/define-*`.

**Phase-by-phase commits.** Each phase writes results as it goes. If interrupted, partial work is preserved. Resuming picks up at the next incomplete phase.

**Onboarding state file.** `_workdesk/onboarding-state.md` tracks per-phase progress:

```yaml
---
started: 2026-04-26
phases:
  environment-check: complete
  role-map: complete
  context-setup: incomplete
  tool-setup: pending
  first-daily-plan: pending
  graduation: pending
---
```

**Idempotent operations.** Re-running a phase is safe — adding `clients/` when it exists = no-op. No destructive actions during onboarding.

**Skip detection via vault-improvements.** If operator never completes onboarding, vault-improvements detects partial state and surfaces *"Resume `/onboarding`?"* via `[REVIEW]` inbox. (Suppressed during the first 14 days, so this fires only if onboarding is genuinely abandoned.)

**Sub-commands:**
- `/onboarding` — resumes from incomplete phase, or runs full flow first time
- `/onboarding --status` — shows what's been configured
- `/onboarding --restart` — wipes state with confirmation (vault content untouched)
- `/onboarding --update-profile` — interactive profile edit, post-graduation

For adding new engagement containers post-onboarding: run `/define-object` directly. The meta-skill recognizes engagement shape from the JTBD interview answers and scaffolds with the 4-item lighter folder format. No separate `--add-engagement` sub-command.

**Confirmation guards.** Running `/onboarding` after graduation says: *"Onboarding complete. Use --restart to redo everything (vault content is safe), or --update-profile to adjust role mix."* No accidental wipes.

### Update path — versioned declarations + 3-way merge (Gap #8 resolution)

**V1 ships these seeds (so V2 can update cleanly later):**
- `version:` field on every shipped artifact (declarations, skills, rules, hook scripts)
- `_workdesk/defaults/` reference copies of every shipped artifact (V1 baseline)
- `_workdesk/snapshots/` directory created (empty, ready for pre-update snapshots)
- Bootstrap logs `bootstrap-install-completed` event to `system/events/{YYYY-MM}.md`

**V2 ships (deferred — built later):**
- `/workdesk-update` skill that performs 3-way merge:
  1. V1 default (baseline) — `_workdesk/defaults/{file}`
  2. V2 default (new) — shipped in update
  3. User's current — `_workdesk/{zone}/{file}`
- For each shipped artifact: auto-apply V1→V2 changes that don't conflict; surface conflicts via `[REVIEW]` inbox (*"keep mine / take V2 / merge"*)
- `/workdesk-update --preview` dry-run mode shows changes without applying
- Auto-snapshot to `_workdesk/snapshots/{date}-pre-v2/` before any merge
- Migration scripts for schema changes (e.g., new frontmatter field on existing notes)
- After merge, update `_workdesk/defaults/` to V2 baseline

**What's sacred (never auto-modified):**
- Vault content: atlas/, gtd/, intel/, system/, personal/
- User-created declarations (not based on a shipped default)
- User customizations within shipped declarations (preserved via 3-way merge)

3-way merge is the established pattern (git, npm, package managers). Snapshots provide rollback. Dry-run prevents surprises.

### Maturity model — bootstrap → guided 14 days → steady state

**Stage 0: Bootstrap.** Desired user outcome: *"The vault is installed and safe."* Bootstrap delivers folder structure, declarations, daily note template, welcome inbox item, `operator-profile.md` stub. Does not try to teach the system. Bootstrap → onboarding bridge: a `[REVIEW]` welcome pointer in `gtd/inbox/` says *"Run `/onboarding` to set up your role mix and contexts. Optional: setup videos at [link]."*

**Stage 1: Onboarding.** Desired user outcome: *"The vault reflects my life enough to use tomorrow."* Six phases above. Captures minimum viable operator model. Self-sufficient — videos optional.

**Stage 2: Guided first 14 days.** Desired user outcome: *"This system is helping before it starts optimizing itself."* Active behaviors:
- daily-plan adds short contextual guidance ("you have 3 active areas; weekly-review will surface stale ones")
- weekly-review **active from week 1** — this is the bridge skill
- vault-improvements **suppressed for 14 days** (no patterns yet)
- recurring items can be suggested, never auto-created without confirmation
- Empty zone scans return *"this is empty, here's what would go here"*

Key shape: **weekly-review on, self-improvement off**. Stable everyday loop ships before the system starts modifying itself.

**Stage 3: Steady state.** Desired user outcome: *"The vault keeps up with my work without creating admin drag."* Triggers:
- onboarding complete
- at least one weekly-review generated
- at least one of: project, initiative, recurring item, or processed transcript exists

When all three trigger, `operator-profile.md` flips `first-30-days-mode: graduated`. Tutorial framing drops; vault-improvements activates; daily-plan tonality matures.

User flow: install → `/onboarding` → daily use + weekly-review → steady-state graduation. Videos enrich but don't gate.

### Platform scope — Mac-only V1 (Gap #6 resolution)

V1 ships exclusively for macOS. Linux probably works (similar Unix) but isn't officially supported. Windows users see a *"V2 will support Windows"* message if they try to install.

**Mac-only enables:**
- `_workdesk/` real directory + `.claude` symlink works natively (no Developer Mode required)
- Hook scripts ship as bash in `_workdesk/scripts/` (no Node.js cross-platform layer)
- Unix paths throughout — no separator conversion
- Single shell-script bootstrap installer
- Optional connectors only — V1 succeeds with **zero mandatory third-party integrations**

**The vault content architecture is OS-agnostic** — declarations, meta-skills, signals are platform-neutral. Only the install/runtime layer is Mac-specific. V2 adds Windows support by replacing the install/runtime layer; vault content unchanged.

**Avoid hidden dependency sprawl.** V1 must not depend on Node-based bootstrap tooling, background daemons, OS-level launch agents, or non-standard package managers. Shell + small bash helpers only — every moving part testable and recoverable.

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
EMPTY-VAULT means: nothing outside { .obsidian/, .DS_Store, .git/, .gitignore,
                                      a single empty README.md if user pre-created one }
if vault has content beyond that allow-list:
  warn + refuse install
  point to manual-migration documentation
else:
  proceed with fresh install
```

**Greenfield-only is strict in V1.** Existing `.claude/`, `system/log.md`, or any other content outside the empty-vault allow-list causes bootstrap to refuse with the migration message. V1 does **not** auto-migrate prior installs — that's V2's `/migrate` skill. Operators with existing content either start a fresh vault or wait for V2.

**Bootstrap edge cases (environmental, must handle gracefully — not migrations):**
- **Vault on iCloud Drive** — `_workdesk/` is a real directory so it survives iCloud eviction better than a symlinked target. The `.claude` symlink may still be evicted; bootstrap warns: *"iCloud-backed vault detected; if Claude Code reports missing `.claude/`, run `brctl download _workdesk/` or move the vault out of iCloud."*
- **Vault on Obsidian Sync** — Obsidian Sync syncs `_workdesk/` (real directory) but skips the `.claude` symlink. On other devices, recreate the symlink with `ln -s _workdesk .claude` from the vault root, or skip if Claude Code isn't installed there.
- **Permission failures** — `chmod +x` on hook scripts may fail on some filesystems (NFS, FAT). Bootstrap retries once, then errors with explicit recovery steps.

**V2 ships migration assistant (deferred):**
- `/migrate` skill that detects existing content, maps to WorkDesk OS structure
- Walks operator through proposed moves (per file, per folder)
- Executes with full backup snapshots before any change
- Not in V1.

## Critical Files To Be Created

### Bootstrap installer
- `bootstrap.sh` — Mac shell script. Responsibilities (in order):
  1. Verify macOS
  2. Verify target vault is empty per the allow-list
  3. Create the five-zone folder skeleton
  4. Create `_workdesk/` structure (real directory)
  5. Install hooks and declarations
  6. Create `.claude` symlink → `_workdesk/` (compatibility alias)
  7. Seed welcome `[REVIEW]` in `gtd/inbox/`
  8. Write `bootstrap-install-completed` event to `system/events/{YYYY-MM}.md`
  9. Run post-install self-check

### Post-install self-check
Bootstrap finishes by verifying:
- All required directories exist
- `personal/` read-only protection is active (test write fails)
- Hooks are reachable and within latency budget
- `.claude` symlink resolves correctly
- Write access works in non-personal zones

If self-check fails, install stops in a recoverable state and emits a plain-language repair note. No silent failures.

### Five zone folders (created by bootstrap, mostly empty on day 1)
- `personal/daily/` (the one universal practice)
- `atlas/meetings/`, `atlas/decisions/`, `atlas/people/`, `atlas/initiatives/`, `atlas/areas/`
- `gtd/inbox/`, `gtd/actions/{next,waiting}/`, `gtd/projects/`, `gtd/recurring/{schedules,checklists}/`, `gtd/someday/{actions,projects}/`, `gtd/archive/{projects,actions}/`
- `intel/briefings/{daily,weekly}/`, `intel/vault-improvements/`, `intel/research/`, `intel/observations/`
- `system/intake/`, `system/transcripts/`, `system/session-log/`, `system/events/`, `system/media/`

### `_workdesk/` infrastructure (real directory; `.claude` is a symlink to it)
- `_workdesk/operator-profile.md` — role mix, contexts, enabled tools, daily-planning style
- `_workdesk/onboarding-state.md` — tracks per-phase onboarding completion
- `_workdesk/settings.json` — declares PostToolUse hook for `system/events/`, PreToolUse hook for `personal/` lock, Stop hook for learnings
- `_workdesk/scripts/post-tool-use-log.sh` — semantic-event hook script (10 classes)
- `_workdesk/scripts/pre-tool-use-personal-lock.sh` — read-only enforcement
- `_workdesk/scripts/bench-hooks.sh` — verifies p95 < 50ms
- `_workdesk/scripts/bootstrap-vault.sh` — vault-content scaffolding helper
- `_workdesk/skills/` — V1 core skills:
  - `/onboarding` (six phases + `--status`, `--restart`, `--update-profile`)
  - `/daily-ops` (invokes daily-plan)
  - `/weekly-review` (invokes weekly-review)
  - `/extract`
  - `/obsidian-markdown`
  - `/pobo` (with `--lite` flag)
  - `/promote-to-project`
  - `/checklist` (materialize a recurring checklist as actions)
  - `/create-spec`
  - `/codex-rescue`
  - `/define-object`, `/define-signal`, `/define-source`, `/define-practice`, `/define-tool`, `/define-rule`
- `_workdesk/rules/` — universal rules (no-fabrication, source-documentation, matching, double-entry-knowledge, writing-style, claude-md-coevolution, per-project-accounting) + universal tool refs (obsidian-cli, qmd, defuddle)
- `_workdesk/objects/` — declarations for action, project, recurring, inbox-item, meeting, decision, person, initiative, **area**
- `_workdesk/signals/` — declarations for daily-plan, **weekly-review**, vault-improvements (each with `## Learnings` section seeded empty + `version: 1.0`)
- `_workdesk/sources/` — declarations for transcript, session-log, intake
- `_workdesk/practices/` — declaration for daily
- `_workdesk/defaults/` — reference copies of every shipped artifact (V1 baseline for future V2 merge)
- `_workdesk/snapshots/` — empty, ready for pre-update snapshots
- `_workdesk/templates/` — format scaffolds for meta-skills

### Existing files to reuse / adapt (paths after bootstrap migration; pre-existing content under `.claude/` moves into `_workdesk/`)
- `_workdesk/hooks/stop-learnings.sh` — already exists, reused for `## Learnings` section + skill learnings.md scanning
- `_workdesk/rules/per-project-accounting.md` — already exists at 8-item structure
- `_workdesk/skills/{obsidian-cli,qmd,defuddle,pobo,daily-ops,extract,obsidian-markdown}/` — already exist, reused
- `_workdesk/skills/onboard-client/` — adapted into the broader `/onboarding` skill or kept as a sub-flow
- `_workdesk/agents/orchestrator.md` — kept; routes between skills
- `system/log.md` (existing 91 lines in Khalil's current vault) — V1 does **not** migrate this; it lives only in the existing vault and bootstrap refuses to install over it (greenfield-only). V2's `/migrate` skill will translate prior `log.md` entries into `system/events/{YYYY-MM}.md` format when migration ships.

## Critical Risks And Mitigations

Stress-testing the plan against six personas (consultant, founder, employee, researcher, creative, parent), Mac implementation realities, and the bootstrap → onboarding → daily-use lifecycle surfaced these concrete risks. Each is mitigated in the design above; this section makes them visible.

### Risk 1 — Empty-vault value gap
**Problem:** a fresh vault feels sterile and over-engineered.
**Mitigation:** ship `areas/` universal, ship `recurring/` first-class, ship the daily note template, ship the 7-step daily-plan fallback chain so a sparse vault still produces useful output.

### Risk 2 — Consultant bias
**Problem:** V1 reads as if all meaningful work is client-shaped.
**Mitigation:** `areas/` is universal and required; engagement containers are optional overlays selected during onboarding's role-map phase. Personal-life and employee users get a clean day-1 fit without scaffolding `clients/`.

### Risk 3 — Initiative explosion
**Problem:** users create too many multi-folder contexts; especially parents and employees whose work is mostly recurring maintenance.
**Mitigation:** stricter initiative test (bounded outcome only); default to area + recurring-or-action unless a real outcome exists; lite-POBO path for low-stakes projects; default-to-action when the project test is ambiguous.

### Risk 4 — Hidden control plane
**Problem:** users cannot inspect or trust the system they're meant to operate.
**Mitigation:** `_workdesk/` is the visible source of truth; `.claude` is the compatibility symlink. Operators see the control plane in Obsidian without plugins.

### Risk 5 — Hook fragility
**Problem:** all-event logging creates noise, performance drag, and failure points.
**Mitigation:** narrow to 10 semantic event classes; monthly event files (no rotation hook); `shlock`-based concurrency; 50ms p95 latency budget enforced via bench script.

### Risk 6 — Optional tool lock-in
**Problem:** users without GWS or transcript tools get a degraded product that feels broken.
**Mitigation:** zero mandatory third-party integrations; `enabled-tools` in `operator-profile.md` gates per-signal source attempts; missing connectors are optional, not failures; daily-plan and weekly-review degrade gracefully across rich/sparse/offline.

### Risk 7 — Overlong onboarding
**Problem:** setup becomes training theater.
**Mitigation:** six tight phases; no JTBD interview on day 1; advanced modeling moves to normal use via `/define-*`; videos optional, not required.

### Risk 8 — Inbox flood
**Problem:** every detection drops `[REVIEW]` items; operator drowns.
**Mitigation:** per-session cap of 7 new items + batched roll-up beyond that; ≥0.7 confidence threshold; backlog signal at 20 unresolved; auto-expiry by prefix.

### Risk 9 — Link rot on archive
**Problem:** polymorphic `parent:` links and `[[wikilinks]]` break when targets archive or rename.
**Mitigation:** agent-driven moves rewrite inbound links in the same operation; operator-driven moves get caught by vault-improvements' broken-link scan.

## Acceptance Criteria

V1 passes if a fresh user can experience all four:

### Universality
A consultant, founder, employee, researcher, creative, and parent each complete onboarding without needing a custom zone or hidden schema change. Each gets a day-1 vault that fits their role mix.

### Mac feasibility
A fresh macOS install completes with:
- one bootstrap command
- one self-check
- zero mandatory third-party integrations

### Sparse-data usefulness
A user with no calendar integration, no transcript integration, and only three manually created notes still gets a coherent daily-plan and weekly-review.

### Structural clarity
The operator can answer without opening implementation docs:
- where does enduring responsibility live (`atlas/areas/`)
- where does recurring work live (`gtd/recurring/`)
- where do projects live (`gtd/projects/` or `atlas/initiatives/`)
- where do interpretations live (`intel/`)
- where do raw inputs live (`system/`)

## Build Order

This order matters. V1 should not ship self-improvement before it ships a stable everyday loop.

1. Bootstrap skeleton + post-install self-check
2. Personal lock enforcement (PreToolUse hook)
3. Atlas core types **including `areas/`**
4. GTD core types **including `recurring/`**
5. Operator profile + 6-phase onboarding
6. Daily-plan with sparse-data fallback chain
7. Weekly-review (active from week 1)
8. Transcript processing
9. Event logging (`system/events/{YYYY-MM}.md`, 10 semantic classes)
10. Vault-improvements (suppressed first 14 days)

## Open Questions (Deferred Until V1 Telemetry)

1. **`/extract` capture mechanism** — read Claude Code's transcript file at session-end, or hook each turn? Decide before build kicks off.
2. **Mobile capture flow** — Obsidian on iOS/iPadOS doesn't run Claude Code. Mobile drops to `system/intake/` are silent until next Mac session. Acceptable for V1; revisit if delay creates problems.
3. **Tags vs zones** — V1 default: tags allowed, no tag-based signal logic. Revisit in V1.x if operators ask.
4. **POBO friction** — lite-POBO mitigates. If telemetry shows projects rarely created, revisit the POBO ritual itself.
5. **Inbox triage UX** — V1 ships `gtd/inbox/` as a folder. Single triage view (Obsidian Bases) deferred to V1.x.
6. **Multi-device divergence** — separate `.claude/` per machine; not addressed in V1, documented as known-not-supported.
7. **Sensitive content** — no encryption story; vault-level security is operator's responsibility (FileVault).

### Cross-persona coverage check

| Persona | Day-1 fit | Notable gap |
|---|---|---|
| Consultant | Strong — `areas/` + optional `clients/` | None significant |
| Founder | Strong — `areas/` + optional `businesses/` | Investor/cofounder/customer subtypes emerge via vault-improvements |
| Employee | Strong — `areas/role`, `areas/career`, optional `teams/` | 1:1 cadence handled via `recurring/`; performance reviews emerge via `/define-object` |
| Researcher | Strong — `areas/methods`, `areas/teaching`, optional `labs/` | Citation/literature-management deferred; papers via `/define-object` |
| Creative | Strong — `areas/studio`, `areas/audience`, optional `disciplines/` | Versioned drafts and visual references via `/define-object` |
| Parent / personal | Strong — `areas/household`, `areas/family`, etc. + `recurring/` | Daily-plan tonality work-shaped until V1.x personal-mode templates |

## Verification

End-to-end smoke test on a fresh Mac (or fresh test vault):

### 1. Bootstrap test
- Point bootstrap at empty test vault → installs cleanly + post-install self-check passes
- Point bootstrap at vault with existing content → refuses gracefully with migration message
- Point bootstrap at vault with existing `.claude/` directory → refuses install (greenfield-only; not a V1 migration target)
- Verify `_workdesk/` is visible in Obsidian without "Show Hidden Files" plugin
- Verify `.claude` symlink resolves to `_workdesk/`
- Verify `personal/` is hard-locked (try to write via Claude Code → blocked by PreToolUse hook)
- Verify hook latency under 50ms p95 via `_workdesk/scripts/bench-hooks.sh`

### 2. Onboarding test (six phases, mixed persona)
- First Claude Code session → finds welcome `[REVIEW]` in `gtd/inbox/`
- Run `/onboarding` with role mix `[consultant, parent]` → all six phases complete
- Verify `_workdesk/operator-profile.md` reflects role mix and enabled-tools
- Verify `atlas/areas/` instances created (e.g., `family/`, `household/`) and `atlas/clients/` engagement container created
- Verify `_workdesk/onboarding-state.md` updates per-phase
- Interrupt mid-Phase 3 → re-run → resumes at Phase 3 (idempotent)
- Run `/onboarding --status`, `--restart`, `--update-profile` — each behaves correctly

### 3. Object lifecycle test
- Drop a meeting transcript in `system/transcripts/` → processes into `atlas/meetings/{date}-{slug}.md` + creates/updates `atlas/people/`, `atlas/decisions/` as appropriate
- Verify source frontmatter on transcript flips `processed: true` with `processed-into:` backlinks
- Verify `system/events/{YYYY-MM}.md` records `source-processed` + `object-created` lines
- Verify above-threshold creations (≥0.7 confidence) drop `[REVIEW]` pointers in `gtd/inbox/` (and confirm 7-per-session cap holds)
- Verify sub-threshold (<0.7) detections stay silent or ask inline — never drop a `[REVIEW]`

### 4. Signal test — three data conditions
- **Rich data:** run `/daily-ops` with calendar + transcripts + active areas → produces full daily-plan
- **Sparse data:** disable connectors, leave only 3 manual notes → daily-plan still produces useful output via fallback chain
- **Cold start:** empty vault post-bootstrap → daily-plan produces setup-oriented plan, not hollow summary
- Run `/weekly-review` end of week 1 → produces `intel/briefings/weekly/{date}-weekly-review.md` with proposed closures, promotions, cleanup
- Verify vault-improvements is suppressed for first 14 days, fires on day 15

### 5. GTD lifecycle test
- `/pobo --lite` produces 3-field stub project; `_brief.md`, `_status.md`, empty `plan.md`, full 8-item folder
- Full `/pobo` produces project with populated `_brief.md`, `_status.md`, `plan.md`
- Promote next physical action → creates `gtd/actions/next/{slug}.md` with `parent:` link
- Move action to `gtd/actions/waiting/` → status updates work
- Move action to `gtd/archive/actions/{year-month}/` → archive structure works
- `/promote-to-project {action-slug}` upgrades an action to a full project folder

### 6. Recurring lifecycle test
- Create `gtd/recurring/schedules/weekly-payroll.md` with `cadence: weekly`, `next_due: 2026-05-01`
- weekly-review surfaces it as due
- Promote to action → `gtd/actions/next/weekly-payroll-2026-05-01.md` with `parent:` link
- Mark action complete → `next_due` rolls to 2026-05-08; `action-completed` event logged
- `/checklist {slug}` materializes `gtd/recurring/checklists/publish-workflow.md` items as a sequence of actions

### 7. Meta-skill test
- Run `/define-object` for an emergent type (e.g., `companies/`) → JTBD interview produces declaration in `_workdesk/objects/` + scaffolded folder
- Run `/define-signal` → produces declaration with anchors + traversal + output-format + `## Learnings` section seeded empty
- Run `/define-tool` → adds tool reference rule + permission updates

### 8. Hook + log test
- Verify PostToolUse hook only logs the 10 semantic event classes (not every Write/Edit/Bash)
- Verify month rollover: cross midnight on month boundary → next event lands in new `system/events/{YYYY-MM}.md`
- Verify lock contention: parallel writes serialize cleanly, dropped entries warn but don't block

### 9. Link integrity test
- Archive a project → inbound `parent:` links rewrite; archive recorded in `events/`
- Manually drag a project to `gtd/archive/` in Obsidian → next vault-improvements scan flags broken links as `[REVIEW]`

### 10. Codex review (this plan)
- Run `/ultrareview` against this plan file before any implementation begins
- Codex returns confidence rating + adversarial findings
- Address findings in plan before V1 build kicks off

## Source

POBO planning sessions 2026-04-17 → 2026-04-26 with Khalil. Codex adversarial review 2026-04-19. Walkthrough sessions 2026-04-23 → 2026-04-26.
