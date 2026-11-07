# Enable and modes

Nothing is recorded until you turn recording on in a repository. A repository where you never ran `lothal-ledger enable`, and any directory outside a repository, is not recorded.

## Enable

Run `enable` inside the repository you work in. On a terminal it lists the agents it finds on this machine and asks which to add; type their numbers or names, or `all`:

```console
$ cd ~/src/your-project
$ lothal-ledger enable
Agents on this machine:
  1. Claude Code (claude-code)
  2. Codex (codex)
  3. omp (omp)
Add which? Numbers or names, separated by spaces or commas; `all` for every one not added; Enter for none: all
{"slug":"claude-code",…,"status":"installed",…}
…
{"slug":"codex",…,"status":"installed",…,"next_steps":["Codex skips these hooks until they are trusted: start `codex` and choose \"Trust all and continue\" at its \"Hooks need review\" prompt, …}
…
{"slug":"omp",…,"status":"installed",…}
…
{"slug":"git",…,"status":"installed",…}
{"slug":"repository",…,"path":"~/src/your-project",…,"capture":"local",…,"status":"enabled",…,"note":"captured on this machine only: nothing about this repository leaves it, logged in or not",…}
```

It does four things:

1. **Records this repository**, in `local` mode on its first enable (below); a re-run keeps its recorded mode and content setting unless you explicitly change them. The choice is kept in `.git/lothal/settings.json`, which your worktrees of the repository share and which is never committed.
2. **Adds the agents you picked**: their hooks, in each agent's own configuration in your home directory. Only its own entries are added; yours are never replaced or reordered.
3. **Registers the `lothal-ledger` tool server** with those agents, so they can ask the ledger `why`, `dossier` and `search`, and record decisions.
4. **Installs six git hooks** in this repository, only into free slots. A hook that is already there (yours, or a hook manager's such as husky or lefthook) is left alone and reported as `skipped`; commits are still linked later.

The output is one JSON line per agent, one for git and one for the repository; the repository line says what it records and what that means. Without a terminal (a script, CI), `enable` asks nothing; name the agents with `--agent`:

```bash
lothal-ledger enable --agent claude-code --agent codex
```

The agent hooks are installed once for your user. In every further repository you want recorded, run `lothal-ledger enable` there too.

**Codex: trust the hooks once.** Codex runs a hook only after you have reviewed it. Start `codex` in the repository and choose **Trust all and continue** when it lists the new hooks, or trust the `lothal-ledger` entries in its `/hooks` view. Until you do, Codex records nothing. (`lothal-ledger enable --agent codex --trust-codex-hooks` writes the trust for you, for scripted installs.)

**Claude Code and omp** start using the hooks with their next session; a session already running keeps the hooks it started with. In a folder Claude Code has not trusted yet, accept its trust dialog.

**A session already running when you enable.** Its recording starts where its next step finds it; what it wrote before is not recorded. Run `lothal-ledger enable` inside that session (from its shell) and it asks once: `Start here or backfill? [S/b]`. Answer `b` to keep the earlier part too, then record it yourself with `lothal-ledger backfill --session <session>` (the `s_…` reference `enable` printed). It shows what it would record first (nothing leaves your machine for the preview) and records only once you confirm (`--yes` in a script). It records each part once: what was recorded already is left as it is. It follows the same rules as recording itself: your ignore rules, a deleted session stays deleted, an incognito session is never recorded, a `local` repository sends nothing, and a `sync` repository uploads under your current sign-in only. An agent's JSONL transcript can be recorded the same way by its path: `lothal-ledger backfill <transcript> --agent claude-code`. Nothing ever runs it for you.

## The modes

Each enabled repository has a mode, and records everything except what its ignore rules exclude (see [Keep content out](#keep-content-out)):

| `status` says | What is recorded | What leaves this machine |
|---|---|---|
| `local` | sessions, prompts, responses, the model's thinking, tool outputs, the context the model was given, which lines each session wrote | nothing, signed in or not |
| `sync` | the same | while you are signed in: what is recorded from then on, redacted on this machine first (see [Sign in and sync](login-and-sync.md) and [Privacy](privacy.md)) |
| `local`, repository excluded whole (`**`) | attribution and links, with hashes and byte counts instead of excluded text, including explicit decision summaries and rationales | nothing |
| `sync`, repository excluded whole (`**`) | the same | while you are signed in: attribution, links and byte counts, never excluded prose or its unkeyed hashes |

Explicit decisions have no exemption: a rule excluding any file a decision names excludes its summary and rationale too. Existing queued decision text is checked against current exclusions before upload.

A repository is never called "team": the team platform is where a `sync` repository's uploads go. Signing in never changes a repository's mode.

A re-run of `enable`, with or without `--agent`, keeps the repository's mode while repairing agents, hooks and tool entries; only `--local` or `--sync` changes it. A new repository defaults to `local`.

Change a repository's mode by running `enable` again in it with the mode you want:

```console
$ lothal-ledger enable --sync
Agents on this machine:
  1. Claude Code (claude-code)  added
  2. Codex (codex)  added
  3. omp (omp)  added
Every agent found is added; their hooks are brought up to date.
…
{"slug":"repository",…,"capture":"sync",…,"status":"enabled",…,"note":"captured, and uploaded to the team while logged in (`lothal-ledger login`); a login never uploads what was captured before",…}
```

`--local` goes back to `local`:

```bash
lothal-ledger enable --local
```

### Keep content out

Content is kept out by ignore rules, not by a mode: `.lothal-ledger/ignore` in the repository, `.git/lothal/ignore` (beside the repository's `.git/lothal/settings.json`: never committed, read by every checkout of the repository, linked worktrees included), and `ignore` in the user config directory for every repository, all in gitignore syntax, one rule per line, `#` for a comment. An item a rule excludes (a file an edit or a tool call names, and everything when a rule excludes the repository whole: `**`) keeps its hash, its size and the rule that excluded it (`repo:.lothal-ledger/ignore:3`, `repo_state:2` or `user:1`, the file and line), never its content. What was excluded once stays excluded after the rule goes.

The user file is read first, then `.lothal-ledger/ignore`, then `.git/lothal/ignore`, a later line winning, except that no repository file undoes what the user file excludes: a `!` line in `.lothal-ledger/ignore` or `.git/lothal/ignore` lifts only what a repository file excluded, never a path or the whole repository (`**`) that a line of your `ignore` excludes. A repository may add exclusions to what you chose, never remove them; a `!` line in your own file still lifts your own rules.

```console
$ mkdir -p .lothal-ledger && echo '**' >> .lothal-ledger/ignore
$ lothal-ledger status
…
This repository (~/src/your-project): local, content none
…
```

Remove the rule to record content again:

```bash
rm .lothal-ledger/ignore
```

`--content none` and `--content full` were removed in 0.5.0 and are refused. A repository enabled `--content none` before is migrated by the first `lothal-ledger` command you run in it: the command appends `**` to its `.git/lothal/ignore`, which every checkout of the repository reads as it read the old setting (or to the user `ignore` file when that file cannot be written), and says which file and line it wrote. Remove that line to record content from then on.

`content_storage: none` in a settings file (`settings.json` in the user config directory, `.lothal-ledger/settings.json` or `.lothal-ledger/settings.local.json`) is migrated the same way: the next command appends `**` to the user `ignore` file or to the checkout's `.lothal-ledger/ignore`, removes the key and says which file and line it wrote. When that rule would not keep content out (the file is at its size or rule cap, or a later `!**` in `.git/lothal/ignore` lifts it), the file is left as it was, the key stays and keeps content out, and each command says which line stops it. `LOTHAL_LEDGER_CONTENT_STORAGE=none` is refused: every command says to use an ignore rule instead, and no content is recorded while it is set.

`status` lists every repository you enabled with its mode:

```console
$ lothal-ledger status
…
git hooks: current (~/src/your-project/.git/hooks)
Repositories: 1 enabled
  ~/src/your-project: local
This repository (~/src/your-project): local
Sync: not signed in (not logged in): nothing leaves this machine
…
```

## Check the setup

```console
$ lothal-ledger doctor
Doctor: warning
  codex: untrusted
Binary on PATH: "ok"
Hook binary: "ok" ("~/.lothal-ledger/bin/lothal-ledger")
…
Git hooks: current (~/src/your-project/.git/hooks); mode: local
…
Repositories: 1 enabled
  ~/src/your-project: local
This repository (~/src/your-project): local
Codex: "untrusted"
  Codex skips these hooks until they are trusted: …
MCP: ok
  claude-code (user): current
  codex (user): current
  cursor (user): absent
  omp (user): current
…
claude-code: ok (version …, minimum 2.0); user=true repo=false
codex: ok (version …, minimum 0.146); user=true repo=false
…
omp: ok (version …, minimum none); user=true repo=false
…
```

`doctor` only reads; it never changes your agents' configuration. The first line is `Doctor: ok`, or `warning` or `error` with each check that needs attention below it. Look for:

- `Hook binary: "ok"`: the hooks run `~/.lothal-ledger/bin/lothal-ledger`.
- `Git hooks: current` for this repository, or each `skipped` slot with its reason.
- `MCP`: the row of each agent you use reads `current`. `absent` means the agent cannot call the ledger's tools: run `lothal-ledger enable` in the repository; its mode and content setting stay as they were.
- `Codex`: no missing trust. Until you trust the hooks in Codex, `doctor` warns `codex: untrusted` and repeats the step above.
- One line per agent with its version. `version_unverified` means the agent's version could not be read, so its support is not confirmed; it still records.

`lothal-ledger doctor --json` gives the same checks for scripts, and the overall state with what needs attention:

```console
$ lothal-ledger doctor --json | jq -c '{status, attention: [.attention[] | {check, state}]}'
{"status":"warning","attention":[{"check":"codex","state":"untrusted"}]}
```

[Troubleshooting](troubleshooting.md) says what to do about each warning.

## Turn it off

In a repository, `disable` turns recording off there and removes its git hooks. What was recorded stays on your machine, and the agents' hooks stay installed for your other repositories:

```console
$ lothal-ledger disable
{"slug":"git",…,"status":"uninstalled",…}
{"slug":"repository",…,"status":"disabled",…,"note":"nothing is captured here any more; what was captured stays in the local store; the agents' user-scope hooks and MCP entries stay installed for your other repositories",…}
```

Outside any repository, `disable` removes every agent's hooks and tool entries. Run it once you have turned off every repository:

```console
$ cd ~
$ lothal-ledger disable
{"slug":"claude-code",…,"status":"uninstalled",…}
{"agent":"claude-code",…,"status":"removed",…}
{"slug":"codex",…,"status":"uninstalled",…}
{"agent":"codex",…,"status":"removed",…}
…
{"slug":"omp",…,"status":"uninstalled",…}
{"agent":"omp",…,"status":"removed",…}
…
```

`disable` removes exactly what `enable` wrote; entries it did not write are left in place. `--agent <slug>` removes one agent only, wherever you run it.

To pause recording without changing anything, set `LOTHAL_LEDGER_DISABLE=1` in the environment of your agent.

Next: [Sign in and sync](login-and-sync.md).
