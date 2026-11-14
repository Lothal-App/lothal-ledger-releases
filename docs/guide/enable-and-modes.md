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

1. **Records this repository**, in `local` mode on its first enable (below); a re-run keeps its recorded mode unless you name another. The choice is kept in `.git/lothal/settings.json`, which your worktrees of the repository share and which is never committed.
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

**A session already running when you enable.** Its recording starts where its next step finds it; what it wrote before is not recorded. Run `lothal-ledger enable` inside that session (from its shell) and it asks once: `Start here or backfill? [S/b]`. Enter or `s` starts here; without a terminal it starts here without asking. A later `enable` does not ask that session again and never widens what the first answer left out. Answer `b` to keep the earlier part too, then record it yourself with `lothal-ledger backfill --session <session>` (the `s_…` reference `enable` printed). It shows what it would record first (nothing leaves your machine for the preview) and records only once you confirm (`--yes` in a script). It records each part once: what was recorded already is left as it is. It follows the same rules as recording itself: your ignore rules, a deleted session stays deleted, an incognito session is never recorded, a `local` repository sends nothing, and a `sync` repository uploads under your current sign-in only. An agent's JSONL transcript can be recorded the same way by its path: `lothal-ledger backfill <transcript> --agent claude-code`. Nothing ever runs it for you.

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

That includes the snapshots the working log keeps under `.git/lothal` to tell which lines an agent wrote: for an excluded file, they hold a hash of each line instead of the line, and no copy of an edit's text. Its lines are still attributed, line by line: an agent's change to part of a line makes the whole line the agent's. These hashes never leave your machine.

A shell call names a file in its command (`cat secret/.env`, `cd secret && cat .env`, `git show HEAD:secret/.env`) and reads files by a glob or a recursive search (`cat secret/*`, `rg token`); it is excluded when what it reads holds a file or directory a rule excludes, as it is on disk now. A command whose text does not tell what it prints (`git log -p`, `git show`, `git diff`, `cat $FILE`, `xargs cat`, `eval`, `… | sh`) is excluded whenever the repository has any rule that excludes something: it fails closed, because the files it prints may be any file ever committed, deleted ones included. A script or an interpreter prints what its code says, which the command text does not show, and is not judged.

A search call (Claude Code's `Grep`, omp's `grep`) reads everything below its `path` (a path, a `;`-separated list of them or a glob, which reaches the directory it starts in; a link is followed), the working directory when it has none (its `glob` only narrows that); it is excluded when what it reads holds a file or directory a rule excludes, as it is on disk now, and so is anything it reads in `.git`, which holds every committed file: that fails closed whenever the repository has any rule that excludes something. A file-name search (`Glob`, omp's `find`) lists names and is kept.

The user file is read first, then `.lothal-ledger/ignore`, then `.git/lothal/ignore`, a later line winning, except that no repository file undoes what the user file excludes: a `!` line in `.lothal-ledger/ignore` or `.git/lothal/ignore` lifts only what a repository file excluded, never a path or the whole repository (`**`) that a line of your `ignore` excludes. A repository may add exclusions to what you chose, never remove them; a `!` line in your own file still lifts your own rules.

```console
$ mkdir -p .lothal-ledger && echo '**' >> .lothal-ledger/ignore
$ lothal-ledger status
…
This repository (~/src/your-project): local, content none; content kept out by ignore rules: 1 rule in ~/src/your-project/.lothal-ledger/ignore
…
```

`status` names a repository excluded whole `content none`; it is a rule, not a mode, and `status` names each ignore file in force and how many rules it holds. Remove the rule to record content again:

```bash
rm .lothal-ledger/ignore
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
- `MCP`: the row of each agent you use reads `current`. `absent` means the agent cannot call the ledger's tools: run `lothal-ledger enable` in the repository; its mode stays as it was.
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

## Keep a session out: incognito

An incognito session records nothing from the moment you choose it, and stays incognito for good: resuming it, a fork of it and the subagents it starts are not recorded either, even once the variable below is gone. Start a new session to record again.

- **From its start.** Start the agent with `LOTHAL_LEDGER_DISABLE=1` in its environment (for example `LOTHAL_LEDGER_DISABLE=1 claude`), or in omp type `/ledger-incognito` before your first prompt. The session leaves nothing in the ledger: no session, no upload, and its edits read `unattributed`. A commit made from its shell gets no `Lothal-Session` trailer naming it, and the agent cannot record a decision.
- **During a session.** In omp, type `/ledger-incognito`: recording stops at once. When the session recorded something already, omp asks whether to keep it (`Keep what it recorded`) or delete it (`Delete this session from the ledger`, which runs `delete-session` below and shows what it reports: a team copy whose delete is pending; [What the agent tells you](in-agent.md#what-the-agent-tells-you)); unanswered after 60 seconds, it is kept. Until you answer, nothing of the session uploads. Neither answer starts recording again. Resuming a recorded session of another agent with `LOTHAL_LEDGER_DISABLE=1` makes it incognito from then on; nothing asks, and what it recorded before is kept.

## Delete a session

`lothal-ledger delete-session <session>` removes one session from this machine entirely, with its subagent sessions: what they recorded, their links to the lines they wrote, their decisions, their queued uploads and their search entries. `<session>` is the `s_…` reference `sessions` and `why` print. What the agent recorded before the delete is never read in again, also through a fork or subagent of the session the ledger had not read yet; a resumed session, or such a fork, starts afresh. The command ends with `complete: secure delete, full-text index rewrite, free-space scrub and truncating checkpoint done` once nothing of the session is left readable on this machine, including the search index's keys and the free space of the store's pages; if another program kept the store busy it says so, and running it again finishes the delete.

`delete-session` leaves a store of another build, set aside ([A store of another build](storage.md#a-store-of-another-build)), as it is: what was recorded there stays in `~/.local/state/lothal-ledger/set-aside/`. To remove it, delete the `set-aside/` directory.

It deletes on this machine only. When a `sync` repository already uploaded part of the session, the team platform keeps that copy for now: `status` lists the session as `Deleted session <hash>: team delete: pending (Phase 7)`, and a later release deletes it there.

Next: [Sign in and sync](login-and-sync.md).
