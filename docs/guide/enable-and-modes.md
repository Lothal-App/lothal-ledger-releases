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

## The modes

Each enabled repository has a mode and a content setting:

| `status` says | What is recorded | What leaves this machine |
|---|---|---|
| `local` | sessions, prompts, responses, the model's thinking, tool outputs, the context the model was given, which lines each session wrote | nothing, signed in or not |
| `sync` | the same | while you are signed in: what is recorded from then on, redacted on this machine first (see [Sign in and sync](login-and-sync.md) and [Privacy](privacy.md)) |
| `local, content none` | attribution only: which session wrote which lines, with hashes and sizes instead of any text | nothing |
| `sync, content none` | attribution only | while you are signed in: the attribution, hashes and sizes; never a prompt, a response or file content |

A repository is never called "team": the team platform is where a `sync` repository's uploads go. Signing in never changes a repository's mode.

A re-run of `enable`, with or without `--agent`, keeps the repository's mode and content setting while repairing agents, hooks and tool entries. Only `--local` or `--sync` changes the mode, and only `--content none` or `--content full` changes content; each leaves the other setting alone. A new repository defaults to `local` with `full` content.

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

```console
$ lothal-ledger enable --content none
…
{"slug":"repository",…,"capture":"sync, content none",…,"status":"enabled",…,"note":"captured, and uploaded to the team while logged in (`lothal-ledger login`); a login never uploads what was captured before; attribution only: no prompt, response or file content is kept, locally or remotely",…}
```

`--local` goes back to `local`, and `--content full` back to keeping content:

```bash
lothal-ledger enable --local --content full
```

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
