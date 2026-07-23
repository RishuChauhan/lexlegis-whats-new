# Changelog process

Every change ships through a labelled PR so the weekly GitHub Release can be
generated from merge history, then fed to customers (What's New) and staff.

## End-to-end flow (target design)

```
Developer writes code
        │
        ▼
GitHub Pull Request
        │
        ▼
PR follows naming & labeling standards
        │
        ├── feat
        ├── enhance
        ├── fix
        ├── docs
        ├── internal
        ▼
PR Merged
        │
        ▼
Weekly / Version Release
        │
        ▼
Automatic Changelog Generation
        │
        ▼
Categorize changes
        │
 ┌──────┼───────────────┐
 ▼      ▼               ▼
Website  Internal Team   App / Zendesk
```

**Status:** this is the intended end state. What's actually wired up today
(`.github/workflows/pr-check.yml`, `.github/workflows/release-notify.yml`)
implements a subset of it — see "Current implementation vs. target" below.

## Changelog Google Sheet (manual staging)

Between the release email landing and the three-way fan-out, there's a manual
staging step: a shared Google Sheet that acts as the single point of knowledge
for every change before it's pushed out to the Internal Team, `app.lexlegis.ai`,
and the website's What's New page.

Template: [`changelog-sheet-template.csv`](changelog-sheet-template.csv) — import
it into a new Google Sheet (File → Import → Upload) or paste it into an
existing one. Columns:

| Column | Purpose |
|---|---|
| `Release` | The version/release tag this entry shipped in |
| `Date Merged` | When the PR merged |
| `PR` | PR number/link |
| `Title` | PR title (`type(module): summary`) |
| `Type` | `feat` / `enhance` / `fix` / `internal` (/ `docs`, see caveat above) |
| `Module` | `ask`, `interact`, `draft`, `mira`, `library`, `platform`, `experience` |
| `User-Facing` | `yes` / `no`, copied from the PR label |
| `Internal Summary` | Raw/technical description, for the Internal Team channel |
| `External Summary (Customer-Facing)` | Rewritten customer-friendly copy — only needed when `User-Facing` is `yes` |
| `Website Status` / `App/Zendesk Status` / `Internal Team Status` | `Not started` / `Ready` / `Published` per channel, tracked independently since they publish on different cadences |
| `Owner` | Who's writing/publishing the customer-facing copy |
| `Notes` | Anything channel-specific (e.g. Zendesk article link once published) |

Workflow: after each weekly release email lands, add one row per entry, fill in
the external summary for `user-facing: yes` items, then update the per-channel
status columns as each one gets published.

## PR title

`type(module): summary` — e.g. `feat(draft): research before drafting`.
Modules: `ask`, `interact`, `draft`, `mira`, `library`, `platform`, `experience`.

## Required labels (the PR-hygiene check enforces both)

Pick **one type** label and **one user-facing** flag:

| Type | Meaning |
|------|---------|
| `feat` | New feature / launch |
| `enhance` | Improvement to something existing |
| `fix` | Bug fix |
| `internal` | Refactor, chore, infra — no user impact |

| Flag | Meaning |
|------|---------|
| `user-facing: yes` | A customer would notice this change |
| `user-facing: no` | Invisible to customers |

> The diagram above also lists a `docs` type. It is not yet enforced by
> `pr-check.yml` (which currently requires one of `feat, enhance, fix, internal`)
> — treat `docs`-only changes as `internal` + `user-facing: no` until the label
> and hygiene check are extended.

## Flow

1. Branch → open PR with title + the two labels.
2. PR-hygiene check must pass (`.github/workflows/pr-check.yml`).
3. Squash-merge to `main` (squash is the only merge type enabled).
4. Weekly: **Releases → Draft a new release → Generate release notes → Publish**.
5. On publish, `.github/workflows/release-notify.yml` emails the notes.

The **external** cut (customers) takes only `user-facing: yes` entries, rewritten.
The **internal** cut (staff) takes the whole release, lightly grouped.

## Mock demos of this pipeline

To rehearse the label → PR → merge flow without touching real site content:

- Branch name: `demo/pipeline-mock-<date>`.
- Change: a one-line addition under `docs/demo-notes/`, never `index.html` or `styles.css`.
- Labels: always `internal` + `user-facing: no` — a mock run has no user impact by definition.
- Never draft or publish a real GitHub Release as part of a mock demo — that step emails the team and should stay a deliberate, manual action.

## Current implementation vs. target

The diagram's final fan-out — **Website / Internal Team / App or Zendesk** — is
partly defined now (2026-07-23) and partly a future initiative:

### Internal Team — done, unchanged
`release-notify.yml` emails the release notes on every publish. The sheet's
`Internal Summary` column can be pasted in for cleaner grouping, but no new
mechanism is needed here.

### Website — content PR, reusing existing infra
This repo's site (`index.html`) is hand-authored static HTML — no CMS, no data
file, each entry is a hardcoded `<article class="entry">` block. So "publish
to Website" means:

1. Take rows marked `Website Status: Ready` and `User-Facing: yes` from the sheet.
2. Write the `External Summary` copy into a new `<article class="entry">`
   block in `index.html`, matching the existing markup (`data-module`,
   `data-type`, `data-date`).
3. Open a PR for that content change through the normal labelled-PR flow,
   squash-merge to `main`.
4. GitHub Pages redeploys on merge — merged PR = `Website Status: Published`.

This is manual today (someone writes the HTML by hand); worth automating later
if entry volume grows, but there's no CMS to wire up yet.

### App / Zendesk — not built, out of scope for this repo
`app.lexlegis.ai` has **no in-app changelog surface today** — this is a new
feature to design and build in that app's own codebase, not a distribution
step onto existing infra. It needs its own scoping (in-app modal/banner vs.
a Zendesk help-center article vs. both) once that work is picked up. Track it
as a separate initiative; don't block the Website/Internal flow above on it.
