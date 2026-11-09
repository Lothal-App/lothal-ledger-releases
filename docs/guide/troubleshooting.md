# Troubleshooting

## Start with `doctor`

```bash
lothal-ledger doctor
```

`doctor` checks this machine and the repository you run it in, and changes nothing. Its first line is the overall state:

- `Doctor: ok`: nothing needs you.
- `Doctor: warning`: something records less than it could (an agent's version unverified, a git hook slot held by another hook, the store nearly full or recording stopped at its cap); each such check is listed under the first line. The ledger keeps working.
- `Doctor: error`: something is broken, for example the local store cannot be opened. Fix it before relying on what is recorded.

`lothal-ledger doctor --json` gives each check with its `status` and details, for a bug report or a script.

## The round trip

`doctor` also reads back the agents' own records the ledger keeps and compares each with the agent's file or database, at the place it came from. It checks the most recent 256 records (at most 16 MiB); `lothal-ledger doctor --all` checks every one, and `lothal-ledger doctor --session <session>` lists each record of one session. Its `Round trip:` line is followed by one line per agent file, counting its records:

- `identical`: the record is the agent's, byte for byte.
- `identical apart from declared omissions`: the same, except where the ledger declared it left something out (opaque thinking, content an ignore rule excluded, an item over the item cap). With `--session`, each omission is listed with its place and, for an excluded one, its rule. Nothing to do.
- `source changed` or `missing`: the agent rewrote or deleted its own file after the ledger read it. The ledger keeps what it read, and the check stays `ok`. Nothing to do.

Two things are not right and worth a report ([Feedback](feedback.md)), with `doctor`'s output:

- `a new high-entropy field` in a round trip warning: an agent's update added a field no reader of the ledger classifies yet. The ledger keeps only its presence and size.
- `transcript records neither stored nor skipped with a reason` under `transcripts`: the ledger read records it neither kept nor left out for a named reason, a gap it should not have.

## Common problems

**`lothal-ledger: command not found` after installing.** Open a new shell, or run `source "$HOME/.config/lothal-ledger/env.sh"`.

**Nothing is recorded in a repository.** Run `lothal-ledger enable` in it; nothing is recorded until you do. `lothal-ledger status` then says `This repository (…): local` (or its other mode) instead of `not captured`.

**Nothing is recorded from Codex.** Trust the hooks in Codex (see [Enable and modes](enable-and-modes.md#enable)); `doctor`'s `Codex` line names what is missing.

**An agent shows `not_found` or is not listed by `enable`.** Its program was not on your `PATH` when you ran `enable`. Install it, then run `lothal-ledger enable --agent <slug>` again (`claude-code`, `codex`, `omp`). The repository keeps its mode and its ignore rules.

**The agent cannot use the ledger's tools.** `doctor`'s `MCP` rows show `absent` or `outdated` for it: run `lothal-ledger enable` in the repository, then start a new agent session. This repairs the integrations without changing the repository's mode or its ignore rules.

**After an upgrade, a ledger tool answers `MCP server retired after an upgrade`.** The agent session still runs the tool server of the release before. Reconnect the server (in omp, `/mcp reload`) or restart the session ([Install](install.md#upgrade)).

**After upgrading from 0.4.8 or earlier, answers say `legacy store`.** The old store was set aside unchanged and a new one started ([Install](install.md#upgrade)); a line or session only the old store holds is answered from it and says so. Nothing to do: it is read-only and does not count toward the storage cap ([Storage](storage.md#the-store-set-aside-at-an-upgrade)). Until the background flusher has set it aside, `doctor` says `migration pending` and what the hooks capture waits for it.

**`status` says `content none`, or content reads `Not recorded: an ignore rule excludes it.`** An ignore rule excludes it ([Keep content out](enable-and-modes.md#keep-content-out)): `status` lists the ignore files in force and how many rules each holds. A repository excluded whole has a `**` line in `.lothal-ledger/ignore`, in `.git/lothal/ignore` (where the ledger moved a repository's old `--content none` setting) or in `~/.config/lothal-ledger/ignore`. Delete that line to record content from then on; what was recorded while it was there keeps no content.

**A git hook is `skipped`.** Another hook or a hook manager (husky, lefthook, pre-commit, `core.hooksPath`) holds that slot. Nothing of yours is changed. Commits are still linked, when the next agent hook or `lothal-ledger` command runs, as long as git's own records prove which commit each was made on.

**A line reads `unknown`.** This machine has no record of the commit that line came from: it was made on another machine, before you enabled the repository, or in a way git's records cannot tie to its sessions. `unknown` never means a person wrote it, and it is never shown as `unattributed`. Signed in, `why` asks the team platform for such lines.

**A line reads `pending`.** The commit's attribution is still being worked out in the background. Commands wait up to 2 seconds for it; run the command again a moment later.

**A command prints `lothal-ledger: team platform: …; local answer only`.** Signed in, an explicit CLI command (`why`, including `--all-users`, `--remote` and remote `--context`; `sessions`; `search`; `trace`) gives its team platform reads one budget of up to 10 seconds, even when an agent runs the command in its shell. MCP tools, hooks and dossier injection keep a 150-millisecond fail-open budget so they do not hold up the agent's turn. On a slow or flaky connection, or when the team platform is down, the reader answers from this machine alone and says so in that one line on its error output. Nothing is lost; ask again later, or use the CLI when a deliberate read needs more time than MCP allows.

**The session line says `sign-in expired`, or uploads stopped.** Run `lothal-ledger login` again ([Sign in and sync](login-and-sync.md#sign-in)); what was recorded meanwhile uploads afterwards.

**`sync --backfill-local` refuses.** It uploads under your sign-in, so it needs one:

```console exit=1
$ lothal-ledger sync --backfill-local
lothal-ledger: sign in first (`lothal-ledger login`): the backfill queues captures under the current login
```

**The store is full: `not recording: budget`.** The store reached its capture allowance (the cap less its maintenance reserve), so recording stopped whole: nothing new is recorded or attributed until there is room. Raise `storage.cap`, delete sessions you do not need with `lothal-ledger delete-session`, shorten retention, or free disk space when the disk is short; recording resumes on its own once 1 % of the cap is free. See [Storage](storage.md#when-the-store-is-full).

**Anything else.** Run `lothal-ledger doctor` and report what it says ([Feedback](feedback.md)). The hooks' own diagnostics are in `~/.local/state/lothal-ledger/logs/hook.log`; they never contain prompt or response text.

`LOTHAL_LEDGER_DISABLE=1` in an agent's environment does not pause recording: it makes each session it reaches incognito for good ([Keep a session out: incognito](enable-and-modes.md#keep-a-session-out-incognito)). To stop recording a repository, run `lothal-ledger disable` in it.

## Remove everything

Turn recording off in every repository you enabled, then remove the agents' hooks and tool entries from outside any repository:

```bash
cd ~/src/your-project
lothal-ledger disable
cd ~
lothal-ledger disable
```

What was recorded stays until you delete it. The first line also removes the store an upgrade set aside (`legacy/`); the second, the repository's shared `.git/lothal/ignore`:

```bash
rm -rf ~/.local/state/lothal-ledger
rm -rf ~/src/your-project/.git/lothal
```

A repository's `.lothal-ledger/` directory (its settings and ignore rules) is part of the repository; remove it there if nobody needs it.

Then the programs and the installer's files (keep `~/.config/lothal-ledger/settings.json` and `~/.config/lothal-ledger/ignore` if you want your settings and ignore rules back on a later install):

```bash
rm -rf ~/.lothal-ledger
rm -f ~/.config/lothal-ledger/lothal-ledger-receipt.json ~/.config/lothal-ledger/env.sh ~/.config/lothal-ledger/env.fish
```

Last, delete the line `. "$HOME/.config/lothal-ledger/env.sh"` the installer added to `~/.profile`, `~/.bashrc` and `~/.zshrc`.

Next: [Feedback](feedback.md).
