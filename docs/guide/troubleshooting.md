# Troubleshooting

## Start with `doctor`

```bash
lothal-ledger doctor
```

`doctor` checks this machine and the repository you run it in, and changes nothing. Its first line is the overall state:

- `Doctor: ok`: nothing needs you.
- `Doctor: warning`: something records less than it could (an agent's version unverified, a git hook slot held by another hook, the store nearly full); each such check is listed under the first line. The ledger keeps working.
- `Doctor: error`: something is broken, for example the local store cannot be opened. Fix it before relying on what is recorded.

`lothal-ledger doctor --json` gives each check with its `status` and details, for a bug report or a script.

## Common problems

**`lothal-ledger: command not found` after installing.** Open a new shell, or run `source "$HOME/.config/lothal-ledger/env.sh"`.

**Nothing is recorded in a repository.** Run `lothal-ledger enable` in it; nothing is recorded until you do. `lothal-ledger status` then says `This repository (…): local` (or its other mode) instead of `not captured`.

**Nothing is recorded from Codex.** Trust the hooks in Codex (see [Enable and modes](enable-and-modes.md#enable)); `doctor`'s `Codex` line names what is missing.

**An agent shows `not_found` or is not listed by `enable`.** Its program was not on your `PATH` when you ran `enable`. Install it, then run `lothal-ledger enable --agent <slug>` again (`claude-code`, `codex`, `omp`). The repository keeps its mode and content setting.

**The agent cannot use the ledger's tools.** `doctor`'s `MCP` rows show `absent` or `outdated` for it: run `lothal-ledger enable` in the repository, then start a new agent session. This repairs the integrations without changing the repository's mode or content setting.

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

**The store is full.** `doctor`'s `Storage:` line says how full and what to do; see [Storage](storage.md).

**Anything else.** Run `lothal-ledger doctor` and report what it says ([Feedback](feedback.md)). The hooks' own diagnostics are in `~/.local/state/lothal-ledger/logs/hook.log`; they never contain prompt or response text.

To pause recording everywhere without changing anything, set `LOTHAL_LEDGER_DISABLE=1` in the environment your agents run in.

## Remove everything

Turn recording off in every repository you enabled, then remove the agents' hooks and tool entries from outside any repository:

```bash
cd ~/src/your-project
lothal-ledger disable
cd ~
lothal-ledger disable
```

What was recorded stays until you delete it:

```bash
rm -rf ~/.local/state/lothal-ledger
rm -rf ~/src/your-project/.git/lothal
```

Then the programs and the installer's files (keep `~/.config/lothal-ledger/settings.json` if you want your settings back on a later install):

```bash
rm -rf ~/.lothal-ledger
rm -f ~/.config/lothal-ledger/lothal-ledger-receipt.json ~/.config/lothal-ledger/env.sh ~/.config/lothal-ledger/env.fish
```

Last, delete the line `. "$HOME/.config/lothal-ledger/env.sh"` the installer added to `~/.profile`, `~/.bashrc` and `~/.zshrc`.

Next: [Feedback](feedback.md).
