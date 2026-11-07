# Lothal Ledger: the alpha user guide

Lothal Ledger records how AI coding agents build your software: which lines each agent wrote, in which session, with which model and from which prompt, and what you and the agent decided along the way. Months later, anyone can ask why a line or a function exists and get that answer back, from the shell, from inside the agent, or from the team's copy.

This guide is for alpha testers. It assumes nothing about the project's source: follow the pages in order on a fresh machine and you go from installing to sharing what your agents recorded. Every command shown here except Ubuntu's own `apt-get` line is run by a test in the repository (installing and updating, which need the network, in a fresh Ubuntu container), and every terminal capture is what that command printed on a fresh machine (your times, versions and paths differ; `…` marks the parts that do).

For readability, captures abbreviate your home directory as `~`. `status`, `doctor` and the terminal UI's Overview actually print absolute paths; the session notice and `sessions` shorten them. Both forms name the same repository. The Claude Code examples replay a session with a model and a vendor prompt ID; model names and turn IDs vary in your own sessions.

## What the alpha is

- A command-line program, `lothal-ledger`, for Ubuntu 24.04 on x86_64. v0.5.1 ships x86_64 Linux only; there is no macOS or Windows download in this release. It hooks into your coding agents and into git, and keeps what it records on your machine.
- Tested every release with **Claude Code**, **Codex** and **omp**. Other agents (Cursor CLI, Copilot CLI, Gemini CLI, OpenCode, pi, Amp, Droid, Windsurf, Antigravity) are supported on a best-effort basis.
- Opt-in per repository: nothing is recorded until you run `lothal-ledger enable` in a repository.
- Local first: without signing in nothing leaves your machine. Signed in, a repository you enabled with `--sync` uploads what it records, redacted on your machine first, to the team platform (the dev stage, `api.dev.lothal.app`).

## What the alpha is not

- It never says a person wrote a line. A line is written by an agent (`ai`), by an agent and then changed (`ai, changed since`), by a tool an agent ran (`tool`), or nothing was observed writing it (`unattributed`). When this machine has no record of a commit, the answer is `unknown`, never `unattributed`.
- It records no keystrokes, no editor saves and nothing outside the repositories you enabled.
- It never blocks your agent or git: when something goes wrong it records less, and says so in `lothal-ledger doctor`.
- It is an alpha: commands and output can change between releases. Upgrades keep your data.

## The pages

| Page | What you do there |
|---|---|
| [Install](install.md) | Install, check, upgrade, and where things live. |
| [Enable and modes](enable-and-modes.md) | Turn recording on in a repository, choose `local` or `sync` and `content none`, check the setup, turn it off. |
| [Sign in and sync](login-and-sync.md) | Sign in, see what uploads and when, upload what you recorded before signing in, sign out. |
| [Privacy](privacy.md) | What is recorded, what leaves your machine, who can read what you uploaded, and what you see instead of text you may not read. |
| [Storage](storage.md) | How much the ledger keeps on your machine, the cap and the reserve, and what happens when it is full. |
| [The terminal UI](tui.md) | See this machine's state, what wrote each line of a file, a line's whole story and the sessions, in a terminal. |
| [VS Code](vscode.md) | Line marks, a why card on hover and a summary above each function in VS Code, Cursor or Windsurf. |
| [Zed](zed.md) | The same in Zed and other editors through the ledger's editor server. |
| [Ask from the shell](cli.md) | Who wrote each line, why a line or a function exists, how the agent got there (the trace), the sessions, search, the JSON for scripts, and the links an editor opens on the web. |
| [Inside the agent](in-agent.md) | What the agent tells you, the card it gets when it reads a file, and the ledger's tools it can call. |
| [On the web](web.md) | Without the CLI: from a repository to why a line exists, a session's whole trace, investigating lines, decisions, search, and the organisation's settings, and what you see without content access. |
| [Troubleshooting](troubleshooting.md) | What `doctor` says and what to do about it; removing everything. |
| [Feedback](feedback.md) | How to report a problem or an idea. |

`lothal-ledger --help` lists every command, and `lothal-ledger <command> --help` gives each one's full reference.

## Words this guide uses

- **Repository mode**: `local` (recorded on this machine only) or `sync` (recorded, and uploaded while you are signed in). A repository is never "team": the team platform is where `sync` uploads go.
- **Content**: recorded in full except named ignore rules. Excluded items keep hashes and byte counts, not text; explicit decision summaries and rationales have no exemption (see [Privacy](privacy.md)).
- **Session**: one conversation with an agent, named `s_…` in the ledger's output.

## Reporting

Found a bug, a confusing message or a missing feature? [Feedback](feedback.md) says where to report it and what to include.
