# Varis Plan Tracker

A Claude Code skill that turns a multi-package plan into a live progress page.

When Claude carries out a plan with several packages, phases or pull requests,
this skill publishes a private claude.ai artifact and keeps it current while
the work runs. The page shows:

- overall progress, weighted by package size;
- the landing queue: which branch is merging now and which wait, in the order
  they should land (the one blocking the most other packages first);
- a dependency graph ("what blocks what"); hover a package to trace its chain;
- every package grouped by workstream, with its state, note and PR links;
- what needs you: access or credentials Claude cannot get itself, and the open
  questions only you can answer.

The page is a fixed template. All content lives in the artifact's own small
database, so each update is a tiny data write, not a new page. Anyone you
share the link with sees the current state.

## Requirements

- Claude Code signed in to claude.ai, with artifacts available. The skill
  uses the `Artifact` and `ArtifactData` tools; without them it tells you so
  and keeps the progress in a local Markdown file instead.

## Install

### From the Claude plugin directory (review pending)

The plugin has been submitted to Anthropic's plugin directory and is waiting
for review. Once it is listed, install it from the **Discover** tab in
`/plugin` in Claude Code, or from [claude.ai/directory](https://claude.ai/directory).
There is no marketplace to add first. Until then, use the marketplace below.

### From the True North Consulting marketplace

```
/plugin marketplace add True-North-Consulting/varis-context-optimization
/plugin install varis-plan-tracker@true-north-consulting
```

The `true-north-consulting` marketplace lives in the
[varis-context-optimization](https://github.com/True-North-Consulting/varis-context-optimization)
repository and lists both plugins.

## Use

Ask Claude to implement a plan with several packages, or say "track this
plan". Claude breaks the plan into packages, publishes the tracker, gives you
the link and updates it as each package moves.

## What it runs, reads and writes

The plugin contains one skill and one HTML template. It runs no hooks and no
scripts of its own.

- **Writes** a copy of the template for each plan, in a folder you choose
  (by default `~/.claude/plan-runs/<plan>/`).
- **Publishes** that page as a private artifact on your claude.ai account and
  stores the plan's packages, states, PR numbers and open questions in the
  artifact's database. Only you and the people you share it with can open it.
- **Loads** two web fonts from Google Fonts when the page is viewed.

Do not put secrets or credentials in a plan's notes: they end up on the page.

## License

MIT. See [LICENSE](LICENSE).
