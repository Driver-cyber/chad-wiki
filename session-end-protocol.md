---
version: 1.0.0
last-updated: 2026-05-02
---

[← map](./map.md)

# Session-end protocol

> The canonical body of Chad's end-of-session ritual. Each project's `CLAUDE.md`
> links here rather than carrying its own copy, so the substance stays
> single-source and the per-project hooks (tracker filename, log path, Inbox
> source, commit tag) live with each project. If anything below is unclear,
> return to `map.md`.

## What this is

A short, opinionated checklist that fires at the end of a working session. It
sweeps the project's Inbox (if there is one), brings the tracker up to date,
appends what was learned to the log, and commits everything to `main` so the
next session — and any deploy targets — pick up the new state.

The protocol is for any project that has been equipped per
[`project-checklist.md`](./project-checklist.md). The
`/session-end-protocol` skill runs this body programmatically; humans can also
follow it by hand.

## Trigger phrases

Chad invokes the protocol — or pieces of it — with any of these:

| Phrase | Meaning |
|---|---|
| *"shipped X, next Y"* | One priority finished, another starting. Run the full protocol with that framing. |
| *"session-end"* / *"wrap up"* / *"close out"* | End-of-session sweep. Run the full protocol. |
| *"add to backlog: Z"* | Capture an idea without re-prioritizing. Backlog-only update. |
| *"sweep inbox"* | Process Inbox-tagged entries interactively (see step 1). |
| *"sweep all"* / *"session-end automated"* / *"process them yourself"* | Bulk-mode opt-in for ONE sweep. Default reverts to interactive next time. |

## The protocol

### 1. Sweep the Inbox — interactive by default

If the project has an Inbox source defined in its `CLAUDE.md`, read each
active Inbox entry and ask Chad per-item what it should become. Options to
offer:

- **priority** — promote to the tracker's priorities list
- **backlog** — append to the tracker's backlog
- **shipped** — append to the tracker's shipped list (already done, just
  capturing)
- **task** — re-route to the project's `Tasks` category (or equivalent
  to-do bucket)
- **re-route** — move to a different project's category in the same Inbox
  source
- **delete** — drop it

Apply each choice as Chad answers. Do not bulk-process unless Chad explicitly
opts in for that sweep — recognized opt-in phrases are listed above. The
opt-in is per-occurrence; default reverts to interactive on the next sweep.

This step is the load-bearing instance of the
[detect → propose → confirm pattern](./collaboration-patterns.md). If the
project has no Inbox source, skip this step.

### 2. Update the tracker

Edit the tracker's `#tracker-data` JSON block — and *only* the JSON block.
The visual page hydrates from the JSON on load, so editing the static HTML
lists duplicates work and creates drift.

Update as appropriate:

- `priorities` — what's now the top of the build queue
- `backlog` — anything captured this session
- `shipped` — what got finished, in object form:
  `{ "date", "what", "tags"?, "learned"? }`
- `updated` — today's date
- `phase` — only if a phase boundary actually crossed

Each project's `CLAUDE.md` names its tracker filename. The schema is in
[`project-checklist.md`](./project-checklist.md).

### 3. Append to the learned log

Append one entry per meaningful completion or skill acquired this session to
the project's learned log (path specified in the project's `CLAUDE.md`).
Always include `repo` and `tags`. Add the optional enrichment fields
(`mood`, `aha`, `struggle`, `frustration_peak`, `wonder`, `first_ever`,
`intensity`, `energy_in`, `real_world_use`, `curiosity_trail`, `artifact`)
for any that are genuinely true.

Empty fields are better than fabricated ones. Schema in
[`project-checklist.md`](./project-checklist.md).

### 4. Ask Chad one question — not a form

Pick the single question most likely to capture something we'd otherwise
lose. Examples:

- "Anything specific you want noted in the learning log from today?"
- "What's the one mood emoji for this session?"
- "Was there an aha moment I should record?"
- "Did anything almost make you rage-quit today?"

One question, not a survey. If the session was clearly complete and there's
nothing salient to surface, skip the question.

### 5. Commit and push

Stage the changed files, commit with a message in the project's convention
(e.g., `[project-dashboard] — Tasks category for to-do capture | log
updated`), and push to `main` so deploy targets and downstream readers pick
up the new state.

## Per-project hooks

The canonical body above is the same everywhere. The seams between it and a
specific project live in that project's `CLAUDE.md`, which should specify:

| Hook | Example |
|---|---|
| Tracker filename | `project-dashboard-tracker.html` |
| Learned-log path | `learned-log.json` at repo root |
| Inbox source | URL/method to read entries (e.g. a Cloudflare Pages Function), or *"none"* |
| Commit-message tag | `[project-dashboard]`, `[der-hain]`, etc. |
| Deploy target | Cloudflare Pages, App Store, etc. — informs whether `main` push is enough |

If a project's `CLAUDE.md` doesn't specify these, the
`/session-end-protocol` skill should ask before assuming defaults — per
[detect → propose → confirm](./collaboration-patterns.md).

---

## Changelog

- **1.0.0 (2026-05-02):** Initial canonical version. Lifted from the embedded
  Session-End Protocol section in `Driver-cyber/project-dashboard/CLAUDE.md`,
  genericized for broad project usability.
