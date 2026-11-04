# Zed (and other LSP editors)

In Zed the ledger comes through `lothal-ledger-lsp`, an editor server that answers the editor about each line from the `lothal-ledger` program on your machine: a hover card per line, a one-line summary per function, and links to the web. The Lothal Ledger extension for Zed starts it, and registers `lothal-ledger mcp` for Zed's Agent Panel. Neither changes what is recorded, and the server sends no diagnostics: a line an agent wrote is not a problem to fix.

## Install the server

From `lothal-ledger` 0.4.5 on, the installer puts `lothal-ledger-lsp` beside `lothal-ledger` in `~/.lothal-ledger/bin` ([Install](install.md)):

```console
$ lothal-ledger-lsp --version
lothal-ledger-lsp …
```

The extension finds it on the `PATH` of your project's shell, and the server finds `lothal-ledger` on that same `PATH`. When neither that `PATH` nor the `binary.path` setting below has the server, the extension downloads the latest release's server into its own directory, and checks for a newer release each time Zed starts it; offline, the last download serves.

## Install the extension

The extension is not part of this release. The public releases repository holds the programs and this guide, not the extension's source, and the extension is published nowhere else this guide can point you to, so you cannot install it from here yet. Until you have it, Zed shows nothing from the ledger except what you add yourself: the [Agent Panel tools](#the-tools-for-zeds-agent-panel) work from your settings alone.

To ask for the extension, open an issue on the releases repository ([Feedback](feedback.md)). With its directory on your machine:

1. Install Rust through rustup: Zed compiles the extension itself.
2. In Zed (0.205 or later), run **zed: install dev extension** from the command palette and choose the extension's `extensions/zed` directory. The Extensions page then lists **Lothal Ledger** as a dev extension.
3. Open your repository's folder. The server starts for the languages Zed ships.

The rest of this page, up to *The tools for Zed's Agent Panel*, is about the extension once installed. In Neovim, Helix and other editors the server needs no extension ([Other editors](#other-editors)).

Code lenses are off in Zed by default. To see the summary line above each function and class, turn them on in your settings (`"menu"` puts them in the code-action menu instead):

```json
{
  "code_lens": "on"
}
```

Or have it as an inlay hint at the end of the symbol's first line:

```json
{
  "inlay_hints": { "enabled": true },
  "lsp": {
    "lothal-ledger": {
      "initialization_options": { "summaries": "inlay_hints" }
    }
  }
}
```

When the programs are not on the `PATH` Zed sees, give their full paths: `binary.path` is the server, `initialization_options.binary` the `lothal-ledger` it runs.

```json
{
  "lsp": {
    "lothal-ledger": {
      "binary": { "path": "/absolute/path/to/lothal-ledger-lsp" },
      "initialization_options": { "binary": "/absolute/path/to/lothal-ledger" }
    }
  }
}
```

## What it shows

Hover a line the agent wrote in the earlier pages (`retry.py`, line 1): the card names the state, the agent, the prompt it was asked, the session and the commit.

```text hover=retry.py:1
● AI · claude-code · claude-opus-5-5

Asked: add a retry budget constant, RETRY_BUDGET = 3, in a new file retry.py

Session s_… · Write · …

Commit … Add a retry budget
```

- **Hover** on a line: its card. The state with its label and mark (the same five as everywhere: AI ●, AI, changed since ◐, Tool · *name* ▲, Unattributed ○, No record ·), the agent and model, the prompt of the turn that wrote the line or the sentence saying why there is none (the same sentences as in the [terminal UI](tui.md#when-the-prompt-is-not-shown)), the session and the commit, the decisions, and links to the web when the repository has them. No record (`unknown`) is never shown as unattributed. In a buffer with unsaved edits, the card is the edited line's and says to save the file for the full story.
- **Code lens** (or inlay hint) above each function and class in Rust, Python, TypeScript, TSX, JavaScript and Go: one line such as `ai 2 · ai~ 1 · claude-code · claude-opus-5-5 · 1 decision`. Clicking it opens the symbol's history on the web, or shows its card.
- **Code actions** on a line, when the repository has web links: open the line's story, the session's trace, or the symbol's chain on the web.
- In a repository where you never ran `lothal-ledger enable`, the hover says *Not enabled in this repository*.

## The tools for Zed's Agent Panel

The extension registers `lothal-ledger mcp` as the context server `lothal-ledger`, so Zed's agent can ask the ledger `why`, `dossier` and `search`, and record decisions. Zed is retiring extension-registered context servers; the same entry in your settings works without the extension:

```json
{
  "context_servers": {
    "lothal-ledger": { "command": "lothal-ledger", "args": ["mcp"] }
  }
}
```

## Other editors

The server speaks the Language Server Protocol over standard input and output, so any editor with an LSP client can start `lothal-ledger-lsp`: in Neovim, `vim.lsp.config('lothal_ledger', {cmd = {'lothal-ledger-lsp'}})`; in Helix, a `[language-server.lothal-ledger]` entry with `config = { summaries = "inlay_hints" }` (Helix has no code lenses).

## Troubleshooting

- **The extension does not start**: **zed: open log** says why. *lothal-ledger-lsp was not found on this worktree's PATH, and downloading it from the releases failed: …* means Zed's shell `PATH` does not have `~/.lothal-ledger/bin` and the download did not work (offline, or a release before 0.4.5, which has no server): add the directory to that `PATH`, or set `binary.path` as above.
- **The hover says *Update lothal-ledger***: the `lothal-ledger` the server found is older than 0.4.5, which brings the commands the server reads; update it ([Install](install.md)). There is no fallback to older commands.
- **The hover says *lothal-ledger is not installed***: the server found no `lothal-ledger`; install it ([Install](install.md)) or set `initialization_options.binary`.
- **No summary lines**: turn code lenses on (above); a language outside the six has hovers only.

Next: [Ask from the shell](cli.md).
