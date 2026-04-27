# WorkDesk OS V1 — Codex Refinement Plan

## Purpose

This plan refines `plans/workdesk-os-v1.md` before implementation. It keeps the locked five-zone architecture, but stress-tests it against six real user archetypes:

- consultant
- founder
- employee
- researcher
- creative
- parent

The goal is not to expand scope. The goal is to make V1:

- universal enough for mixed-persona users
- feasible to ship on macOS without hidden implementation traps
- structurally complete inside the five-zone model
- resilient across the bootstrap → onboarding → daily-use lifecycle

## Codex Position

The five-zone model is viable, but the current V1 plan is under-specified in four places:

1. It assumes most users organize life around named engagements.
2. It lacks a first-class answer for recurring commitments, household operations, and mixed-role users.
3. It overestimates how much hidden infrastructure a fresh user can absorb on day 1.
4. It treats the Mac bootstrap as a shell-script problem when it is really an install-state, permissions, and recoverability problem.

This revision keeps the same five zones and the same unit-per-zone discipline, but adds the missing control points.

## Non-Negotiables

These stay locked in V1:

- Five zones only: `personal/`, `atlas/`, `gtd/`, `intel/`, `system/`
- One primary unit per zone
- `personal/` remains read-only to the agent
- Atlas remains evidence-first; Intel remains interpretation-first
- GTD remains the canonical home for action management
- V1 remains Mac-first and greenfield-first

## Main Changes From The Prior Plan

1. Introduce an explicit operator profile and role map in `_workdesk/`.
2. Ship one universal enduring-context container in Atlas: `areas/`.
3. Treat engagement containers as optional overlays, not the primary universal abstraction.
4. Add a first-class recurring commitments model in GTD.
5. Narrow V1 source ingestion to what can actually be supported on Mac without brittle integrations.
6. Replace the visible `_workdesk/ -> .claude/` symlink idea with the inverse: `_workdesk/` is the visible source of truth, `.claude/` is the compatibility alias if needed.
7. Reduce onboarding dependence on external videos; onboarding must stand on its own.
8. Add a staged maturity model: day 0 bootstrap, day 1 onboarding, days 2-14 guided use, steady state.
9. Reduce hook ambition in V1; log fewer, higher-value events rather than every possible mutation.
10. Add explicit failure-handling for partial installs, missing tools, empty vaults, and low-data users.

## Refined Five-Zone Model

### Zone 1: Personal

No structural change.

V1 rules:

- `personal/` stays read-only to the agent.
- Only `personal/daily/` ships by default.
- Additional practices remain opt-in.

Codex refinement:

- Onboarding must not depend on optional practices existing.
- Daily-use flows must still work if the user never adds journal, reading, or notes practices.

### Zone 2: Atlas

Atlas is where universality was weakest. The prior plan made engagement containers do too much work. That fits consultants and founders, but it is less natural for employees, parents, some researchers, and many creatives.

#### Required universal Atlas types

These should ship in V1:

```text
atlas/
  meetings/
  decisions/
  people/
  initiatives/
  areas/
```

Why `areas/` must ship:

- consultant: client-independent areas like finance, health, admin, pipeline
- founder: company-independent areas like hiring, runway, legal, ops
- employee: role, career, team health, manager relationship, admin
- researcher: program areas, teaching, methods, lab ops
- creative: craft areas, audience, publishing, studio ops
- parent: household, school, childcare, health, family logistics

`areas/` is the universal durable container for ongoing responsibility that is not best modeled as a client, business, lab, or initiative.

#### Engagement containers become optional

These remain valid, but they are created only when the operator actually works this way:

- `clients/`
- `businesses/`
- `teams/`
- `departments/`
- `labs/`
- `collaborations/`
- `disciplines/`
- other user-defined containers

Rule:

- If the context is named, ongoing, and relationship-shaped, use an engagement container.
- If the context is enduring but not relationship-shaped, use `areas/`.

This removes the bias toward consultant/founder mental models.

#### Initiative rule tightened

The current initiative definition is mostly sound, but V1 needs a stricter test:

- Use `atlas/initiatives/` for multi-session work with a real outcome and external or cross-context coordination.
- Do not create initiatives for every ongoing concern.
- Household and role maintenance usually live in `areas/` plus recurring GTD actions, not initiatives.

This prevents initiative sprawl, especially for parents and employees.

#### Atlas object shapes

Keep two shapes:

1. atomic note
2. context folder

But V1 should standardize `areas/` as a context folder:

```text
atlas/areas/{slug}/
  _brief.md
  _status.md
  notes/
  _archive/
```

This is the same light 4-item shape already proposed for engagement containers. Reuse it.

### Zone 3: GTD

GTD is the right home for action management, but the current plan is missing recurring work.

#### Add recurring commitments

V1 GTD should ship:

```text
gtd/
  inbox/
  actions/
    next/
    waiting/
  projects/
  recurring/
    checklists/
    schedules/
  someday/
    actions/
    projects/
  archive/
```

Rationale:

- consultant: weekly client prep, invoicing, pipeline review
- founder: payroll, finance review, hiring loops, board prep
- employee: 1:1 prep, weekly planning, expense/admin tasks
- researcher: reading rotation, paper review, teaching prep
- creative: publishing cadence, review/edit cycles
- parent: school prep, meal planning, care logistics, household resets

Without recurring commitments, daily plans either miss important maintenance work or recreate it ad hoc. Both are failure modes.

#### Recurring model

Use two shapes:

- `recurring/checklists/` for repeatable procedures
- `recurring/schedules/` for cadence-bound commitments

Example:

```yaml
---
type: recurring
shape: schedule
status: active
cadence: weekly
next_due: 2026-05-01
parent: "[[atlas/areas/household]]"
source: ""
---
Weekly school logistics review.
```

V1 behavior:

- recurring items do not live in `actions/next/` permanently
- a due recurring item can be promoted into `actions/next/`
- completion rolls `next_due` forward

This preserves GTD clarity while making the system usable for real life.

#### Parent field remains polymorphic

Keep `parent:` polymorphic. That is one of the stronger parts of the original design.

Add explicit allowed parents in docs:

- `gtd/projects/...`
- `atlas/initiatives/...`
- `atlas/areas/...`
- `atlas/{engagement-type}/...`
- empty

### Zone 4: Intel

The Intel zone is conceptually sound, but the default signals need stronger sparse-data behavior.

#### Ship fewer built-in signal families in V1

V1 default:

```text
intel/
  briefings/
    daily/
    weekly/
  observations/
  vault-improvements/
  research/
```

#### Minimum built-in signals

Ship only:

1. `daily-plan`
2. `weekly-review`
3. `vault-improvements`

Do not require research workflows or custom signal types for V1 success.

#### Daily-plan must degrade across three data conditions

The prior plan handles empty zones too loosely. V1 should explicitly support:

1. rich-data user
2. sparse-data user
3. offline/manual user

Daily-plan fallback order:

1. calendar commitments
2. `gtd/actions/next/`
3. due recurring items
4. active project and initiative statuses
5. today's daily note
6. unread inbox items
7. stale contexts needing attention

If the user has none of that, the system should produce a setup-oriented plan, not a hollow summary.

#### Weekly-review is mandatory

A weekly review signal should ship by default because it is the bridge between onboarding and stable use. It should:

- scan open actions, projects, initiatives, and recurring items
- surface stale contexts
- count inbox backlog
- propose closures, promotions, and cleanup

This is more important for early retention than `vault-improvements`.

### Zone 5: System

System is the most over-ambitious part of the original plan.

#### Narrow raw-source scope in V1

V1 should explicitly support only these source kinds:

- `intake`
- `transcript`
- `session-log`

Optional later:

- bookmarks
- email exports
- screenshots
- PDFs
- OCR-heavy scans

This reduces false promises during the Mac-first launch.

#### Replace `event-log.md` with append-only monthly files

Do not ship a single rolling `event-log.md` plus a rotation hook. Ship:

```text
system/events/
  2026-04.md
  2026-05.md
```

Why:

- simpler implementation
- no rotation race condition
- easier append-only semantics
- easier bounded reads

Event format can stay line-based.

#### Log fewer event classes in V1

Logging every write/edit/bash that touches the vault is too noisy and too fragile for a first release.

Log only:

- bootstrap install completed
- onboarding phase completed
- source processed
- object created
- project/initiative created
- action promoted
- action completed
- signal generated
- declaration changed

This is enough for review signals without building a surveillance-grade hook layer.

## Universal Persona Coverage

### Consultant

Works well with:

- `clients/`
- `initiatives/`
- recurring admin and pipeline reviews
- meetings, decisions, people

Main risk:

- over-creating client-linked initiatives for all work

Mitigation:

- allow direct actions under `clients/`
- reserve initiatives for true multi-session deliverables

### Founder

Works well with:

- `businesses/`
- `areas/` for company-wide functions
- recurring schedules for finance, hiring, ops

Main risk:

- company operations becoming a clutter of faux-projects

Mitigation:

- use `areas/` for standing domains
- use recurring items for operating cadence

### Employee

This persona was underserved in the original plan.

Needed defaults:

- `teams/` optional, not mandatory
- `areas/role`, `areas/career`, `areas/admin`
- recurring 1:1 prep and weekly planning

Main risk:

- the vault feeling too “client work” oriented

Mitigation:

- make `areas/` universal
- make engagement containers optional

### Researcher

Needed defaults:

- `labs/` or `collaborations/` optional
- `areas/teaching`, `areas/methods`, `areas/reading`
- recurring reading and review workflows

Main risk:

- forcing research threads into initiatives too early

Mitigation:

- allow long-lived areas and direct actions
- create initiatives only for bounded outcomes like grant submission, paper revision, experiment push

### Creative

Needed defaults:

- `disciplines/` optional
- `areas/studio`, `areas/audience`, `areas/publishing`
- recurring publish/review cadences

Main risk:

- confusion between personal creative practice and operational creative work

Mitigation:

- personal drafts stay in `personal/`
- shipped work and factual assets land in Atlas
- interpretation and editorial synthesis land in Intel

### Parent

This persona was the weakest fit in the original plan.

Needed defaults:

- `areas/household`, `areas/family`, `areas/health`, `areas/school`
- recurring schedules for family logistics
- direct actions linked to areas, not forced into projects

Main risk:

- the vault becoming unusable because too much life work is recurring, shared, and non-project-shaped

Mitigation:

- recurring GTD support
- `areas/` as a universal durable context

## Mac Implementation Plan

### Supported environment

V1 should explicitly target:

- macOS only
- fresh Obsidian vault
- Claude Code installed
- no required external connectors for core success

Optional connectors:

- Google Workspace
- Granola or transcript source

Core rule:

- if optional tools are missing, the vault must still work

### `_workdesk/` and `.claude/`

The original plan made `.claude/` primary and `_workdesk/` the visible symlink. Reverse that.

Preferred V1 structure:

```text
_workdesk/      # visible, user-facing source of truth
.claude -> _workdesk
```

Why:

- users can see and inspect the control plane immediately
- Obsidian does not need hidden-file support for core understanding
- implementation remains compatible with tools that expect `.claude/`

If Claude Code requires a physical `.claude/` directory rather than a symlink, then bootstrap should mirror required files instead and document `_workdesk/` as the editable surface. But V1 should pick one model and test it end-to-end before shipping.

### Bootstrap responsibilities

`bootstrap.sh` should do only these things:

1. verify macOS
2. verify target vault is empty except for `.obsidian/`
3. create the five-zone folder skeleton
4. create `_workdesk/` structure
5. install hooks and declarations
6. create compatibility path for `.claude/` if supported
7. seed one welcome inbox item
8. write install metadata
9. run a post-install self-check

### Post-install self-check

V1 should not trust bootstrap blindly. It should verify:

- all required directories exist
- read-only protection for `personal/` is active
- hooks are reachable
- `.claude` compatibility path resolves correctly
- write access works in non-personal zones

If self-check fails, install should stop in a recoverable state and emit a plain-language repair note.

### Avoid hidden dependency sprawl

Do not make V1 depend on:

- Node-based bootstrap tooling
- background daemons
- OS-level launch agents
- non-standard package managers

Shell plus small helper scripts is fine, but only if every moving part is testable and recoverable.

## Bootstrap → Onboarding → Daily Use

### Stage 0: Bootstrap

Desired user outcome:

- “The vault is installed and safe.”

Do not try to teach the whole system here.

Bootstrap output should include:

- folder structure
- declarations
- one example daily note template
- one welcome inbox item
- one `operator-profile.md` stub

### Stage 1: Onboarding

Desired user outcome:

- “The vault reflects my life enough to use tomorrow.”

The current plan underweights identity and overweights training. Onboarding should collect the minimum viable operator model.

#### Add operator profile

Ship:

```text
_workdesk/operator-profile.md
```

Fields:

- name
- role mix
- primary contexts
- enabled tools
- preferred naming
- daily planning style
- week start day
- first-30-days mode status

This file is what makes mixed personas workable. A user can be founder + parent + researcher at the same time.

#### Onboarding phases

V1 onboarding should be:

1. Environment check
2. Role map
3. Context setup
4. Tool setup
5. First daily plan
6. Graduation

Phase details:

1. Environment check
   Confirm install health and explain what is already in place.
2. Role map
   Capture mixed persona profile, not just one archetype.
3. Context setup
   Create `areas/` first, then optional engagement containers.
4. Tool setup
   Detect optional connectors; do not block if absent.
5. First daily plan
   Generate a useful output from whatever data exists.
6. Graduation
   Explain the next two actions only: use tomorrow, run weekly review.

#### Onboarding should be low-friction

Hard limits:

- no more than one major conceptual choice per phase
- no long JTBD interview on day 1
- no requirement to watch videos before use

Videos can remain helpful, but onboarding must be self-sufficient.

### Stage 2: Guided first 14 days

Desired user outcome:

- “This system is helping before it starts optimizing itself.”

During first-30-days mode:

- `daily-plan` adds short contextual guidance
- `weekly-review` is active from week 1
- `vault-improvements` is suppressed for 14 days
- recurring prompts can be suggested, but not auto-created without confirmation

Key change:

- weekly review stays on
- self-improvement stays mostly off

This is a better maturity curve.

### Stage 3: Steady state

Desired user outcome:

- “The vault keeps up with my work without creating admin drag.”

Steady-state triggers:

- onboarding complete
- at least one weekly review generated
- at least one of: project, initiative, recurring item, or processed transcript exists

## Critical Risks And Mitigations

### Risk 1: Empty-vault value gap

Problem:

- a fresh vault can feel sterile and over-engineered

Mitigation:

- ship `areas/`
- ship recurring support
- ship first daily note template
- ship a useful first daily-plan even with sparse data

### Risk 2: Consultant bias

Problem:

- V1 reads as if all meaningful work is client-shaped

Mitigation:

- make `areas/` universal and required
- make engagement containers optional

### Risk 3: Initiative explosion

Problem:

- users create too many multi-folder contexts

Mitigation:

- stricter initiative test
- default to area + recurring/action unless a bounded outcome exists

### Risk 4: Hidden control plane

Problem:

- users cannot inspect or trust the system they are meant to operate

Mitigation:

- visible `_workdesk/`
- compatibility alias for `.claude/`, not the other way around

### Risk 5: Hook fragility

Problem:

- all-event logging creates noise, performance drag, and failure points

Mitigation:

- log only high-value semantic events
- use monthly event files

### Risk 6: Optional tool lock-in

Problem:

- users without GWS or transcript tools get a degraded product that feels broken

Mitigation:

- design daily-plan and weekly-review to succeed without connectors
- detect and report missing tools as optional, not as failures

### Risk 7: Overlong onboarding

Problem:

- setup becomes training theater

Mitigation:

- keep onboarding to profile, contexts, tools, first plan
- move advanced modeling to normal use via `/define-*`

## Revised V1 Deliverables

### Required folder skeleton

```text
personal/
  daily/

atlas/
  meetings/
  decisions/
  people/
  initiatives/
  areas/

gtd/
  inbox/
  actions/
    next/
    waiting/
  projects/
  recurring/
    checklists/
    schedules/
  someday/
    actions/
    projects/
  archive/

intel/
  briefings/
    daily/
    weekly/
  observations/
  vault-improvements/
  research/

system/
  intake/
  transcripts/
  session-log/
  events/
```

### Required control-plane files

- `_workdesk/operator-profile.md`
- `_workdesk/settings.json`
- `_workdesk/objects/`
- `_workdesk/signals/`
- `_workdesk/sources/`
- `_workdesk/practices/`
- `_workdesk/rules/`
- `_workdesk/templates/`
- `_workdesk/onboarding-state.md`

### Required shipped declarations

Objects:

- action
- project
- recurring
- inbox-item
- meeting
- decision
- person
- initiative
- area

Signals:

- daily-plan
- weekly-review
- vault-improvements

Sources:

- intake
- transcript
- session-log

Practices:

- daily

## Acceptance Criteria

### Universality

A consultant, founder, employee, researcher, creative, and parent should each be able to complete onboarding without needing a custom zone or hidden schema change.

### Mac feasibility

A fresh macOS install should complete with:

- one bootstrap command
- one self-check
- zero mandatory third-party integrations

### Sparse-data usefulness

A user with no calendar integration, no transcript integration, and only three manually created notes should still get a coherent daily plan and weekly review.

### Structural clarity

The operator should be able to answer:

- where does enduring responsibility live
- where does recurring work live
- where do projects live
- where do interpretations live
- where do raw inputs live

without opening implementation docs.

## Build Order

1. Bootstrap skeleton and self-check
2. Personal lock enforcement
3. Atlas core types including `areas/`
4. GTD core types including `recurring/`
5. Operator profile and onboarding
6. Daily-plan sparse-data path
7. Weekly-review
8. Transcript processing
9. Event logging
10. Vault-improvements

This order matters. V1 should not ship self-improvement before it ships a stable everyday loop.

## Final Recommendation

Proceed with the five-zone architecture, but only if V1 is re-scoped around three universal anchors:

1. `areas/` for enduring responsibility
2. `recurring/` for real-life maintenance work
3. `operator-profile.md` for mixed-persona context

Without those three additions, the system will feel elegant in theory but biased in practice toward consultant/founder workflows and high-data users.
