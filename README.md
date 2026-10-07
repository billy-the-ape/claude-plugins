# claude-plugins

A [Claude Code](https://code.claude.com/docs) plugin marketplace with two plugins:

| Plugin | Skill | What it does |
| --- | --- | --- |
| `planner` | `/planner:plan` | Work out how to implement a change before writing any of it |
| `reviewer` | `/reviewer:review` | Review a diff, branch, or pull request |

The marketplace is named **`ape-plugins`**. That name comes from `name` in
`.claude-plugin/marketplace.json`, not from this repository, and it is what people
type after the `@`.

## Install

```bash
claude plugin marketplace add billy-the-ape/claude-plugins
claude plugin install reviewer@ape-plugins
claude plugin install planner@ape-plugins
```

Or inside a session: `/plugin marketplace add billy-the-ape/claude-plugins`.

## Use from GitHub Actions

`plugin_marketplaces` is newline-separated, so Anthropic's marketplace can stay
alongside this one:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    plugin_marketplaces: |
      https://github.com/anthropics/claude-code.git
      https://github.com/billy-the-ape/claude-plugins.git
    plugins: "reviewer@ape-plugins"
    prompt: "/reviewer:review --comment ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
    claude_args: >-
      --allowedTools "mcp__github_inline_comment__create_inline_comment,Read,Grep,Glob,Bash(git:*)"
```

`--allowedTools` is the entire pre-approved set rather than an addition to a default
one, and a workflow has nobody to approve a prompt — so a review that is allowed only
to post comments cannot read the diff it was asked to review.

## Updates

Neither `plugin.json` sets a `version`, and that is deliberate. Users get a new copy
of a plugin only when its computed version changes, so a pinned version means pushing
to this repository changes nothing for anyone until the string is bumped. With
`version` omitted, installs track commits instead.

How that reaches each consumer:

- **GitHub Actions** installs fresh on every run, so it always gets the current
  default branch.
- **Interactive sessions** cache. Background auto-update is off by default and
  `marketplace.json` has no field to turn it on — each user enables it under
  **Marketplaces** in `/plugin`, or runs `/plugin marketplace update ape-plugins`.

If you later want pinned releases, add `version` to `plugin.json` (not to the
marketplace entry — setting both is an error) and bump it on every release.

## Develop

```bash
claude plugin validate .                     # marketplace + every plugin manifest
claude --plugin-dir ./plugins/reviewer       # load one plugin for a session
claude --plugin-dir ./plugins                # load both
```

Don't point `--plugin-dir` at the repository root: it does not read
`marketplace.json`, so nothing under `plugins/` loads and no error is printed.

Edits to a marketplace added from a local path take effect at the next session start
or on `/reload-plugins`, with no version bump.

## Layout

```
.claude-plugin/marketplace.json   the catalog; this directory makes the repo root
                                  the marketplace root
plugins/<name>/
  .claude-plugin/plugin.json      the manifest — the ONLY file that belongs in here
  skills/<skill>/SKILL.md         one directory per skill
```

Components saved inside `.claude-plugin/` are silently ignored. Optional per-plugin
directories — `agents/`, `hooks/hooks.json`, `.mcp.json` — are siblings of
`.claude-plugin/`, at the plugin root.

Two rules that break installs quietly: a plugin entry's `name` must equal the `name`
in that plugin's `plugin.json`, and a relative `source` is written from the
marketplace root and may not contain `..`.
