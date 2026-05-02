---
version: 1.0.0
last-updated: 2026-05-02
---

[← map](./map.md)

# Project checklist

> What every Chad-project needs to be considered "fully equipped" — meaning
> the [session-end protocol](./session-end-protocol.md) can run cleanly
> without setup work or guesswork. The `/session-end-protocol` and
> `/new-project-bootstrap` skills read this file to decide whether to run,
> retrofit, or scaffold. If anything below is unclear, return to `map.md`.

## Required artifacts

A project is fully equipped when all four of the following are present.

| Artifact | Why required |
|---|---|
| `CLAUDE.md` containing a Session-End Protocol section that links to [`session-end-protocol.md`](./session-end-protocol.md) and names the per-project hooks | The protocol body is canonical here; the project's wiring lives in its own `CLAUDE.md`. |
| A tracker file (`*-tracker.html`) containing a `<script id="tracker-data" type="application/json">…</script>` block matching the schema below | Single source for priorities, backlog, shipped. The visual page hydrates from JSON. |
| `learned-log.json` at the repo root, an array of entries matching the enriched schema below | Append-only record of completions and skills. Drives any cross-project Galaxy / Victory-Lap views. |
| Git remote with a `main` branch that deploy targets and downstream readers track | Push-to-`main` is how state propagates outward. |

## Recommended artifacts

Not strictly required, but most mature projects end up with all three.

| Artifact | When useful |
|---|---|
| `DECISIONS.md` | Any project where architectural calls accumulate. Living log keyed by date and rationale. |
| Inbox source documented in `CLAUDE.md` | If voice/widget capture flows into this project, the protocol's step 1 needs to know how to read it. |
| Deploy target documented in `CLAUDE.md` | If the project deploys somewhere (Cloudflare Pages, App Store, etc.), the protocol's step 5 should mention where push lands. |

## Schema: `#tracker-data` JSON block

```json
{
  "phase": "string — current build phase",
  "updated": "YYYY-MM-DD — last session-end date",
  "columns": ["string", "..."],
  "priorities": [
    "string — the top of the build queue, ordered"
  ],
  "backlog": [
    "string — captured-but-not-prioritized work"
  ],
  "shipped": [
    {
      "date": "YYYY-MM-DD",
      "what": "what shipped",
      "tags": ["string"],
      "learned": "optional"
    }
  ]
}
```

`shipped` items should use the object form (not bare strings) so cross-project
readers can sort by date and filter by tag. Bare strings still render in
older trackers but lose those affordances.

## Schema: `learned-log.json` entry

Required fields:

```json
{
  "date": "YYYY-MM-DD",
  "project": "Display name",
  "repo": "repo-name (matches the project registry)",
  "completed": "What was finished — one sentence",
  "learned": "Skill or concept acquired — one sentence, or null",
  "session_note": "Brief context"
}
```

Optional enrichment (add what's genuinely true; don't pad):

```json
{
  "tags": ["string", "..."],
  "struggle": "What was hard — one sentence",
  "artifact": "path or url — primary file or URL touched",
  "intensity": "quick | session | deep_dive | marathon",
  "mood": "single emoji",
  "aha": "What clicked — one sentence",
  "frustration_peak": "Moment of near-rage-quit — one sentence",
  "curiosity_trail": "Question this opened up — one sentence",
  "first_ever": "Label of a first-time event",
  "real_world_use": true,
  "energy_in": "low | medium | high",
  "wonder": "One sentence on something delightful about how it works"
}
```

Empty fields are better than fabricated ones.

## Detection logic for skills

When the `/session-end-protocol` or `/new-project-bootstrap` skill runs in a
repo, it classifies each required artifact:

- **Present** — file exists *and* satisfies the required structure (e.g. the
  tracker has a `#tracker-data` JSON block; `CLAUDE.md` has a Session-End
  Protocol section).
- **Missing** — file does not exist.
- **Partial** — file exists but doesn't yet meet the structural requirement
  (e.g. a tracker HTML file with no JSON block, or a `CLAUDE.md` with no
  protocol section).

For each non-present artifact the skill **proposes** the canonical scaffold
or retrofit and asks Chad before applying — per
[detect → propose → confirm](./collaboration-patterns.md). Bulk scaffolding
only on explicit opt-in.

A project with all four required artifacts marked Present is "fully equipped"
and the session-end protocol runs without setup work.

---

## Changelog

- **1.0.0 (2026-05-02):** Initial version. Required artifacts, recommended
  artifacts, schema references for tracker and learned-log, detection logic
  for skills.
