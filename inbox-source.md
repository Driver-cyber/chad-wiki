---
version: 1.0.0
last-updated: 2026-05-06
---

[← map](./map.md)

# Inbox source

> The canonical spec for Chad's Inbox: where notes captured from the iOS
> shortcut, widget, and notepad land, and how Claude reads or writes them.
> The session-end protocol's step 1 (Inbox sweep) reads from here; any future
> capture surface that wants to participate in the Inbox writes here. If
> anything below is unclear, return to `map.md`.

## What this is

A single shared backing store for Chad's quick captures across surfaces.
Notes land here from the iOS shortcut (Speak/Type), the iOS widget, the
dashboard's notepad, and any Claude session that re-routes an entry. The
session-end protocol's Inbox sweep is the regular "process the captures"
moment.

The store is a GitHub Gist proxied through a Cloudflare Pages Function on
the project-dashboard deploy — so no token leaks to the client. Reads and
writes go through the proxy, never directly to GitHub.

## Endpoint

`https://derhain.chadstewartcpa.com/api/gist`

The function lives at `Driver-cyber/project-dashboard:functions/api/gist.js`.
It holds `GITHUB_TOKEN` and `GIST_ID` as Cloudflare environment variables
and forwards calls to GitHub's gists API.

## Required client header

Cloudflare's WAF blocks the default `Python-urllib/X.Y` User-Agent. Any
client calling this endpoint must set an explicit User-Agent — anything
browser-like or app-like works:

```
User-Agent: Mozilla/5.0 chad-mac
```

Without it, requests get blocked by the WAF before reaching the function.
This is the most common failure mode for sessions reading or writing the
Inbox programmatically. If a sweep fails with a generic Cloudflare error
page, the User-Agent is the first thing to check.

## File and shape

The gist contains one file: `garden-notes.json`. Its content (after JSON-
parsing the gist response twice, see "Reading" below) is a JSON array of
note objects:

```json
[
  {
    "id": "n_mosveiey18j3pa",
    "project": "Inbox",
    "text": "raw note body",
    "createdAt": "2026-05-05T16:56:24.250Z",
    "status": "active",
    "source": "shortcut-speak"
  }
]
```

| Field | Required | Notes |
|---|---|---|
| `id` | yes | `n_<random>`. Generated client-side at capture; preserved across moves and edits. |
| `project` | yes | One of: `Inbox` (unprocessed capture), `Tasks` (general to-do), `__tbd__` (uncategorized), or a registered project name from `projects.json` (e.g. `garden-app`, `cadence`, `ORDOBook`, `wild-stewart-homeschool`, `project-dashboard`). |
| `text` | yes | Raw body of the note, untransformed. |
| `createdAt` | yes | ISO 8601 UTC timestamp from the moment of capture. |
| `status` | yes | `active` or `archived`. Archived notes stay in the file as historical record — see "Archive, don't delete" below. |
| `source` | recommended | Where the note came from: `shortcut-speak`, `shortcut-type`, `widget`, `notepad`, etc. Helps later analysis. |

## Reading the Inbox

GET the endpoint, then unwrap the gist response twice:

1. Parse the response body — it's the GitHub Gists API response shape (see
   `https://docs.github.com/en/rest/gists`).
2. The notes file is at `response.files["garden-notes.json"].content` — a
   string.
3. JSON-parse that string to get the notes array.

Then filter for the desired view. For the session-end Inbox sweep:

```python
[n for n in notes if n.get('project') == 'Inbox' and n.get('status') == 'active']
```

For a Tasks view, `Tasks` instead of `Inbox`. For per-project views, use
the project's repo name.

## Writing changes back

PATCH the endpoint with the entire updated `garden-notes.json` content as
a string. The function forwards to GitHub's gists API.

```
PATCH https://derhain.chadstewartcpa.com/api/gist
Content-Type: application/json
User-Agent: <anything non-Python-urllib>

{
  "files": {
    "garden-notes.json": {
      "content": "<stringified JSON array of all notes, with mutations applied>"
    }
  }
}
```

The PATCH body needs the *entire* notes array, not just the changed entries
— GitHub's gists API replaces the whole file content. Mutate in memory,
stringify, send.

### Mutation patterns during a sweep

| Disposition | Mutation on the source entry |
|---|---|
| Re-route to another project | Set `project` to the new project name; `status` stays `active` |
| Move to Tasks | Set `project` to `Tasks`; `status` stays `active` |
| Mark as processed (priority/backlog/shipped applied to a tracker) | Set `status` to `archived` |
| Delete | Set `status` to `archived` (see "Archive, don't delete" below) |

For `priority` / `backlog` / `shipped` dispositions, the actual tracker
write (to the destination project's `*-tracker.html` `#tracker-data` JSON
block) is a *separate* operation — see
[`session-end-protocol.md`](./session-end-protocol.md) step 1. The Inbox
source's job ends with the entry archived.

### Archive, don't delete

The notes array is append-only by convention. "Delete" sets `status` to
`archived` rather than removing the row, so the historical record stays
intact and IDs stay stable. If a row is genuinely garbage (e.g., empty
text from a mis-fired capture), archiving it is still preferred over
removal — it's cheap and keeps the audit trail clean.

### Concurrency

This is a simple last-write-wins store — there's no locking. If two
surfaces write at the same time, the second overwrites the first. In
practice this hasn't been an issue because Chad is the only writer and
writes are bursty, not continuous. If a future capture surface ever
races with the sweep, revisit.

## Companion: trackers don't write here

The Inbox source is for capture only. Once a sweep classifies an entry as
`priority` / `backlog` / `shipped`, that write goes to *that project's*
`*-tracker.html` file's `#tracker-data` JSON block — not back to the gist.
The Inbox source's responsibility ends with the entry's `status` flipped
to `archived` once the corresponding tracker write succeeds.

This split keeps each store doing one job: the Inbox is the *capture
log*; each tracker is the *project state*.

---

## Changelog

- **1.0.0 (2026-05-06):** Initial canonical spec — endpoint, User-Agent
  requirement, JSON shape, read/write methods, mutation patterns,
  archive-don't-delete convention.
