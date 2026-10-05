---
name: plan-tracker
description: Publish and keep updated a live progress tracker artifact for executing a multi-step plan (packages/workstreams with dependencies, PRs, blockers, open questions). Use when the user asks to implement a plan with several packages or phases, asks for a progress tracker / status page / "what blocks what" view of a plan, or says "track this plan".
---

# Plan tracker

A private claude.ai artifact that shows, at a glance, how far a plan's
implementation is: overall weighted progress, the landing queue (what holds the
lock, what waits, in landing order), a dependency graph ("what blocks what",
hover to trace a chain), the package list grouped by workstream with PR links,
and two lists — what needs the user's access or credentials, and the open
questions for the user.

The page is a fixed template; all content lives in the artifact's `db`, so
updates are small `ArtifactData` writes, never republishes.

**Requires** the `Artifact` and `ArtifactData` tools (Claude Code signed in to
claude.ai, with artifacts and their `db` capability available). If they are
not available in this session, tell the user the tracker cannot be published
here and keep the plan's progress in a local Markdown file instead.

## Setting one up

1. Break the plan into packages (one row each). For every package decide:
   `code` (short id, e.g. `P3`), `name`, `short` (≤ 22 chars, for the graph),
   `group` (workstream heading), `order` (sort key; groups appear in order of
   their first package), `size` (`S`|`M`|`L` — weights 1/2/3), `deps` (codes it
   needs landed first), `state`, `prs` (numbers), optional `note` (one line:
   what it is doing or waiting for) and `partial` (0–1, the share already
   landed for a package that lands in several PRs).
   `state` is one of `todo`, `running`, `landing` (done and reviewed, waiting
   to merge), `landed`. "Ready to start" vs "Blocked" is derived from `deps`.
2. Copy `template.html` (in this skill's base directory, next to this file) to
   a durable place outside the repository for this plan — e.g.
   `~/.claude/plan-runs/<plan>/tracker.html` — and replace `{{TITLE}}` (the
   artifact's name: two to four words, e.g. "Checkout Plan Tracker") and
   `{{HEADING}}` (the page heading). Change nothing else in the page.
3. Publish it with the Artifact tool: `icon: "checklist"`, a one-sentence
   `description`, and
   `capabilities: {"db": {"rules": [{"path": "", "read": "view", "write": "admin"}]}}`
   (everyone who can open it reads; only the owner and editors write).
4. Seed the data with **one** `ArtifactData` `batch`: a `set` per package in
   collection `packages` (doc id = the code), and `meta/status` with
   `title`, `subtitle`, `source` (e.g. the plan file path), `prUrl` (e.g.
   `https://github.com/<org>/<repo>/pull/` — PR numbers are appended; omit for
   plain text), `needsLabel` (heading of the access list, e.g. "Needs cloud
   access"), `needsAuth` (list of strings), `questions` (list of strings),
   `updatedAt` (human-readable, with time zone), and `landing`:
   `{lock: {code, branch, since, blocks} | null, queue: [{code, branch, blocks}]}`
   in landing order (`blocks` = transitive dependents).
   Optional `buildLock`: `{holder, cmd, since, priority, asOf}` for a build
   lock that parallel sessions share (`holder` null when free) — a snapshot,
   so always set `asOf`.
   Inline JSON with typographic quotes breaks easily: write each document to a
   JSON file and pass `file_path` per batch entry instead.
5. Do one read with `as_level: "view"` to confirm a viewer sees the rows, then
   give the user the link.

## Keeping it current

- Update the row whenever a package changes state, opens or merges a PR, or
  its blocker changes — right when the report arrives, not in batches at the
  end. Use `update` with `if_version` (the last version you saw; every write
  result returns the new one). Remove a stale note with `{"__delete__": true}`.
- Keep `meta/status.needsAuth`, `questions` and `landing` truthful (update
  `landing` whenever the lock changes hands or a branch joins the queue); bump
  `updatedAt`.
- New package discovered mid-run (e.g. a rebase or port step the plan
  implied): add it with its `deps` and add it to dependents' `deps`.
- Record the artifact URL in the plan run's progress notes (and in a memory,
  if the session keeps one), so a later session — after a quota cut or a
  crash — keeps updating the same tracker instead of creating a new one. From
  another conversation, update data with `ArtifactData` on that URL; to change
  the page itself, `Artifact` `read` it first, then publish with `url`.
- Republish an existing tracker with a newer template only when the user asks.

## Landing order

When packages wait to land one at a time (each needs a fresh rebase and
checks), land them by impact, never first-come-first-served: the branch whose
package has the most transitive dependents in the tracker's `deps` graph goes
first, ties by arrival; fixes nothing depends on go last. Keep the run's state
outside the repository (e.g. `~/.claude/plan-runs/<plan>/`): a deps table
(code, deps, branches), a script that prints the queued branches in that
order, and a lock that only the head of that order may take — every reader of
the queue must use the same order. Show each waiting package's queue position
in its tracker `note`.
