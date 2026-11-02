# Install

## Before you start

- Ubuntu 24.04 on x86_64 and a normal user account. Installing needs no root; only the `apt-get` line below does.
- `curl`, `git` and `jq` (this guide reads JSON output with `jq`):

  ```bash
  sudo apt-get update && sudo apt-get install -y curl git jq
  ```

- At least one coding agent installed, signed in and on your `PATH`: Claude Code (`claude`), Codex (`codex`) or omp (`omp`). Install each by following its own documentation. The ledger finds agents by their programs, so install the agent first, or run `lothal-ledger enable` again after you add one.
- A git repository to work in. This guide uses `~/src/your-project`; use your own.

You need no GitHub account and no token: the programs come from the public repository [`Lothal-App/lothal-ledger-releases`](https://github.com/Lothal-App/lothal-ledger-releases).

## Install

```bash
curl -fsSL https://github.com/Lothal-App/lothal-ledger-releases/releases/latest/download/lothal-ledger-installer.sh | sh
```

The installer checks the SHA-256 of the archive containing `lothal-ledger`, `lothal-ledger-tui` and `lothal-ledger-lsp`. The updater is downloaded separately over HTTPS without a checksum check. It installs these four programs into `~/.lothal-ledger/bin`:

- `lothal-ledger`: the command you type, the program your agents' and git's hooks run, and the server the agents' tools talk to;
- `lothal-ledger-tui`: the ledger full screen in your terminal ([Terminal UI](tui.md));
- `lothal-ledger-lsp`: the editor server your editor's Lothal Ledger extension starts (Zed, Neovim, Helix, JetBrains);
- `lothal-ledger-update`: the updater, which keeps the other three at one version.

It adds `~/.lothal-ledger/bin` to your `PATH` in `~/.profile`, `~/.bashrc` and `~/.zshrc`, so new shells find it. In the shell you installed from, run the line it prints:

```bash
source "$HOME/.config/lothal-ledger/env.sh"
```

## Check

```console
$ lothal-ledger --version
lothal-ledger …
```

It prints the release you installed. Then look at the state of this machine; nothing is recorded yet:

```console
$ lothal-ledger status
claude-code: user=false repo=false
codex: user=false repo=false
cursor-cli: user=false repo=false
cursor-ide: user=false repo=false
copilot-cli: user=false repo=false
gemini-cli: user=false repo=false
opencode: user=false repo=false
pi: user=false repo=false
omp: user=false repo=false
droid: user=false repo=false
amp: user=false repo=false
windsurf: user=false repo=false
antigravity: user=false repo=false
git hooks: not_a_repository
Repositories: none enabled; nothing is captured until `lothal-ledger enable` is run in a repository
Sync: not signed in (not logged in): nothing leaves this machine
Queue: 0 pending, 0 held, 0 dead, 0 withheld
Last upload: never
Flusher: idle
```

One line per supported agent says whether the ledger's hooks are installed for it (`user=` in your home directory; `repo=` in a repository's files, which this guide does not use). The rest is this machine's state: no repository enabled, not signed in, nothing waiting to upload.

## Upgrade

```bash
lothal-ledger-update
```

The updater asks the public repository for the latest release and installs it over the old one, in the same place. Your recorded data, your settings, your agents' hooks and your repositories' choices are kept; the hooks run the new version from their next call. After an upgrade, if `lothal-ledger doctor` shows hooks or tool entries as `outdated` or `absent`, run `lothal-ledger enable` again in each repository you work in. That re-run repairs the integrations and keeps each repository's mode (`local` or `sync`) and content setting (`full` or `none`); only explicit mode or content flags change those choices.

**Restart every open agent session after upgrading**, including after `lothal-ledger-update`. Hooks use the new binary on their next call, but an open session can keep its old, long-running MCP server. Once the new binary upgrades the local store, that old server can fail every ledger tool call with `unsupported store schema` and the schema number. Restarting the agent session starts the new server; repairing the registration with `enable` alone does not replace a server that is already running.

Codex sessions recorded by v0.4.4 keep their old turn split after the upgrade: a trace can still show an extra, prompt-less turn with calls whose output was not recorded. The upgrade preserves those records; it does not reconstruct their turns or recover content that was not captured. New sessions use the new capture behavior.

## Where things live

| What | Where |
|---|---|
| The programs | `~/.lothal-ledger/bin/` |
| Install receipt and the `PATH` scripts | `~/.config/lothal-ledger/` (`lothal-ledger-receipt.json`, `env.sh`, `env.fish`) |
| Your settings (optional) | `~/.config/lothal-ledger/settings.json` |
| What is recorded | `~/.local/state/lothal-ledger/` (`ledger.db`, `objects/`, `logs/`) |
| Which repositories record, and how | `.git/lothal/settings.json` in each repository (never committed) |
| Which lines each session wrote | `.git/lothal/` in each repository |

Removing everything is at the end of [Troubleshooting](troubleshooting.md#remove-everything).

Next: [Enable and modes](enable-and-modes.md).
