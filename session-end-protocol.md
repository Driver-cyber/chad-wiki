---
version: 1.1.0
last-updated: 2026-05-06
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

### 1. Sweep the Inbox — interactive, in batches of 3

Read the active Inbox entries from the project's Inbox source. The
read/write mechanism is canonical at [`inbox-source.md`](./inbox-source.md)
— that doc specifies the endpoint, the required User-Agent header, the
JSON shape, and the PATCH method. If the project has no Inbox source
declared in its `CLAUDE.md`, skip this step entirely.

#### Batch size

Render entries in batches of 3. Each batch gets fully settled before the
next batch is shown.

| Active entries | Batches |
|---|---|
| 0 | Skip step 1 |
| 1–3 | One batch |
| 4+ | Sequential batches of 3, last batch may be smaller |

Numbering inside each batch resets from 1 — no global indexing to
remember.

#### Per-batch header

Each batch opens with a one-line header so Chad always knows where he is:

```
Inbox sweep · batch K of T · N entries total
```

K is the 1-indexed batch number; T is total batches; N is the count of
active entries across the whole sweep.

#### Per-entry rendering

Each entry in a batch is rendered with three labeled fields, in this
exact shape:

```
[N of M] · <id> · created YYYY-MM-DD

NOTE
> "<exact text of the note, quoted verbatim, line-wrapped if long>"

PROPOSE
  project: <destination project repo name, or "(delete)">
  bucket:  <priorities | backlog | shipped | task | (—)>
  why:     <one-sentence rationale>
```

`NOTE` is the captured text, untransformed. `PROPOSE` is the
recommendation as a two-dimensional choice — destination project AND
bucket within that project. `why` is one sentence; if the routing is
self-evident (e.g., the note literally names a project), skip the line
rather than padding it.

When the disposition is `delete`, the project field reads `(delete)` and
the bucket reads `(—)`.

#### Confirmation grammar

After rendering all entries in the current batch, prompt:

```
→ all good? or override per-item like "1: garden-app/priority, 3: cadence/backlog"
```

Accepted forms in Chad's reply:

| Form | Meaning |
|---|---|
| `all good` / `yes` / `ok` / equivalent | Apply all proposals in the batch as-is |
| `1: <project>/<bucket>` | Override item 1's destination + bucket; everything else applies as proposed |
| `2: delete` / `2: re-route to <project>` | Bare disposition shortcuts |
| Comma-separated `1: ..., 3: ...` | Multiple overrides in one reply |

The `<project>/<bucket>` syntax is shorthand. Chad may also reply in
plain prose ("send 2 to garden-app priorities, delete the third") and
the assistant should parse intent rather than reject. The shorthand is
for speed, not a learning curve.

#### Bucket vocabulary (locked)

| Bucket | Meaning |
|---|---|
| `priorities` | Top of the build queue, ordered. The next-thing-I'm-working-on list. |
| `backlog` | Captured but not prioritized. Real, not active focus. |
| `shipped` | Already done — capturing for the historical record. |
| `task` | Re-route to the project's `Tasks` category (or equivalent to-do bucket in the same Inbox source). |
| `re-route` | Move within the Inbox source to a different project's category. |
| `delete` | Archive the entry (per `inbox-source.md`, the file is append-only by convention; "delete" sets `status: archived`). |

#### Apply the choices

Once a batch is confirmed, apply each item:

- For `priorities` / `backlog` / `shipped`: write to the destination
  project's `*-tracker.html` `#tracker-data` JSON block (the
  corresponding array). Then archive the source entry by setting
  `status: archived` per [`inbox-source.md`](./inbox-source.md).
- For `task` or `re-route`: update the source entry's `project` field
  to the new value (`Tasks`, or another project name). Status stays
  `active` — the entry stays alive in its new home.
- For `delete`: set the source entry's `status` to `archived`.

All Inbox-source mutations within a batch should accumulate into a
single PATCH per batch — read once, mutate in memory, write once. The
PATCH semantics and required headers are in
[`inbox-source.md`](./inbox-source.md).

#### Bulk-mode opt-in (unchanged)

If Chad uses any of the bulk-mode trigger phrases — `sweep all`,
`session-end automated`, `process them yourself`, or equivalent — bypass
the per-batch gates and process all entries in one go. The opt-in is
per-occurrence; the default reverts to interactive batching on the next
sweep.

#### Pattern reference

This step is the load-bearing instance of the
[detect → propose → confirm pattern](./collaboration-patterns.md). The
batching and the override syntax are how that pattern adapts when the
items list is longer than working memory.

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

## Running this without the skill

Any Claude surface — local Claude Code on chadsminiEXT, the
`claude.ai/code` browser, Cowork, future hardware before the skill is
reinstalled — can follow this protocol manually by fetching this file
and executing the steps against the current project's hooks. The
`/session-end-protocol` skill is just a thin wrapper that automates the
fetch, the loop, and the per-batch gating.

To run by hand:

1. Fetch `https://chadwiki.chadstewartcpa.com/session-end-protocol.md`
   (raw markdown).
2. Read the project's `CLAUDE.md` for the per-project hooks (tracker
   filename, learned-log path, Inbox source, commit-message tag).
3. Execute steps 1–5 against those hooks, applying the rendering and
   confirmation conventions specified above.
4. For step 1's read/write mechanism, also fetch
   [`inbox-source.md`](./inbox-source.md) — that's where the endpoint,
   User-Agent requirement, and PATCH semantics live.

If the wiki is unreachable, follow the fallback protocol in
[`map.md`](./map.md): surface the question to Chad rather than silently
proceeding.

---

## Changelog

- **1.1.0 (2026-05-06):** Step 1 rewritten with explicit two-dimensional
  disposition (project + bucket), batches of 3, per-batch progress
  header, locked bucket vocabulary, and override-syntax confirmation
  grammar. Pointer added to `inbox-source.md` for the read/write
  mechanism (endpoint, User-Agent requirement, PATCH semantics). New
  "Running this without the skill" section so any surface can execute
  the protocol by hand without the local skill installed.
- **1.0.0 (2026-05-02):** Initial canonical version. Lifted from the embedded
  Session-End Protocol section in `Driver-cyber/project-dashboard/CLAUDE.md`,
  genericized for broad project usability.
