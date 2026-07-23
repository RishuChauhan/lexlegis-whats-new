# Changelog process

Every change ships through a labelled PR so the weekly GitHub Release can be
generated from merge history, then fed to customers (What's New) and staff.

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

