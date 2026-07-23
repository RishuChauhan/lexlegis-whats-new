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

## Current implementation vs. target

The diagram's final fan-out — **Website / Internal Team / App or Zendesk** — is
not yet built. Today, `release-notify.yml` sends one email (the raw release
notes) to a single address on every publish; there's no separate categorization
step and no Zendesk/app delivery. Building that out means deciding, concretely:

- **Website**: does this mean re-publishing the `user-facing: yes` cut to the
  What's New site automatically, or is that still a manual copy from the release?
- **Internal Team**: same email as today, or a separate internal channel (Slack, etc.)?
- **App / Zendesk**: what's the actual delivery mechanism — Zendesk API, a help
  center article, in-app changelog widget?

Until those are answered, treat the three-way fan-out as a proposal, not a
shipped pipeline.
