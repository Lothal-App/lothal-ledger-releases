# VS Code (and Cursor, Windsurf)

The Lothal Ledger extension shows, where you read code, what the ledger recorded about each line: which agent and model wrote it, what it was asked, in which session and commit, and what was decided. It reads the ledger through the `lothal-ledger` program on your machine, and it never changes what is recorded: where something needs a command (enabling a repository, signing in), it shows the command and copies it for you to run in a terminal.

## Install

The extension is not on the marketplace yet: each release attaches it as `lothal-ledger.vsix`. Download the latest one and install it:

```bash
curl -fsSLO https://github.com/Lothal-App/lothal-ledger-releases/releases/latest/download/lothal-ledger.vsix
code --install-extension lothal-ledger.vsix
```

In Cursor or Windsurf, install that file with **Extensions: Install from VSIX…** from the command palette. To update the extension, download and install it again after `lothal-ledger-update`: its version is the release's. The extension needs VS Code 1.90 or later and `lothal-ledger` 0.4.3 or later on your `PATH` ([Install](install.md)); when VS Code does not see your shell's `PATH`, set **`lothalLedger.cliPath`** to the program's full path.

## First run

Open the repository's folder and trust it when VS Code asks: the extension stays off in an untrusted workspace, because reading the ledger runs git in the repository, and a repository's git configuration can run code.

The status bar then says the repository's mode, as `status` does: `Ledger: local`, `Ledger: local · content none`, `Ledger: sync · not signed in`, or `Ledger: sync` while you are signed in, with what is waiting to upload (`Ledger: sync · 7 held · 1 failed`). Click it for the actions: open the status panel, run `doctor`, copy the sign-in command, refresh, turn the line marks off. A repository where you never ran `lothal-ledger enable` reads `Ledger: not enabled`, and its menu copies the `enable` command.

## What it shows

- **Line marks** in the gutter, one per line, in the line's state (below). AI, changed-since and tool lines are marked in the scroll bar too.
- **Hover** on a line: its *why* card. The state and the agent and model; the prompt of the turn that wrote the line (*Asked: …*), or why there is none; the session (`s_…`) and the commit; the decisions recorded about it; links to the panels, with **Trace** on a line a session on this machine wrote ([below](#trace-the-turn-that-wrote-a-line)). In a file with unsaved changes, the card says to save it for the full story.
- **CodeLens** above each function and class: a one-line dossier of the symbol, for example `ai 2 · ai~ 1 · claude-code · claude-opus-5-5 · 1 decision` (two AI lines, one changed since, the agent and model, one decision).
- **Panels**, from the hover's links, the CodeLens, the editor's context menu (**Why This Line?**, **Show Trace of the Writing Turn**, **Show Dossier**) or the command palette (category *Lothal Ledger*): **Line** (the full story of a line), **Trace** (the turn that wrote it, below), **Dossier** (a symbol's card, its writes, decisions and history across renames), **Chain** (**Show Chain Across Teammates**: the symbol's history across sessions and, signed in, your teammates' sessions on the team platform) and **Status** (what `status` and `doctor` say).
- **The Lothal Ledger view** in the activity bar: **Sessions** recorded on this machine (each with **Copy Session Reference**), and **Decisions in This File**.

### The line states

| State | Mark | Hover says |
|---|---|---|
| AI ● | solid bar | An agent wrote this line. |
| AI, changed since ◐ | striped bar | An agent wrote this line, and a later change kept part of it. |
| Tool · *name* ▲ | solid bar | A tool (*name*) an agent ran rewrote this line. |
| Unattributed ○ | hairline | No agent or tool write was observed for this line. |
| No record · | dashed outline | This machine has no record for this line. |

No record (`unknown`) means this machine has no record of the commit the line came from; it never means a person wrote it, and it is never shown as unattributed. Colour is never the only sign: each state has its own mark shape, and its own label in the hover. `lothalLedger.decorations.showUnknown` and `showUnattributed` hide those two marks.

### When the prompt is not shown

| The card says | Why |
|---|---|
| Not recorded: this repository records attribution only. | the repository is `content none` |
| Not kept: a capture budget ran out (the store was full, or redaction ran out of time). | the store was full ([Storage](storage.md)), or redaction ran out of time |
| Kept on the team store; removed from this machine. | uploaded, then removed here to make room |
| Lost: the agent's transcript changed. | it was waiting in the agent's transcript, which changed before it was stored |
| Lost: the agent's transcript is gone. | the same, and the transcript was deleted |
| No prompt was tied to this call. | the agent wrote without a prompt the ledger could tie to it |

A line committed a moment ago can read *Still folding: the ledger is reading this commit. Try again in a moment.* while the ledger works out the commit in the background.
The extension asks the CLI to wait up to 1.5 seconds for a pending fold. If the
answer is still pending, the CLI supplies its cache lifetime; the next hover or
editor refresh after that lifetime asks again. There is no background retry loop.

### Decisions and who delegated the writer

The **Line** and **Dossier** panels list the decisions about the code, as `lothal-ledger why` prints them ([CLI](cli.md)):

- A decision the agent recorded with `record_decision` says where: **recorded in** the agent, the session, the turn and the call (`recorded in claude-code session 7de5…, turn 1, call toolu_01A`). **Open the call in its trace** opens that turn in the Trace panel (`lothal-ledger trace <session> --turn <n>`) with the call marked and scrolled into view; a call the page does not hold says so.
- A question the agent asked you is a **question** row with its outcome (*answered*, *unanswered*, *cancelled*) and the session it was asked in (`asked in s_…`). Each question reads as the CLI writes it: `Q: Which floor should low_stock default to?  selected: 5  · also offered: 3 | per item`, or `typed answer` when you typed one (your words follow as *typed answer*), or `no answer`. Questions are on this machine only; the team platform's decisions carry none.

When the session that wrote the line was a subagent, its prompt is labelled **prompt (subagent)** (its parent's instruction, not yours), and each session that delegated it follows, nearest first: **delegated by** `s_…` (the agent and its agent type), the call that spawned it, and the prompt of that call's turn; or *no spawning call recorded*. A session forked, handed off or continued from another lists that session as **forked from**, **handed off from** or **continued from** `s_…`, where it branched, and the prompt of the turn it branched from.

## Trace: the turn that wrote a line

The hover's **Trace** link, **Show Trace of the Writing Turn** in the editor's context menu, or the panel's **Trace** tab opens the **trace** of the turn that wrote the line, as `lothal-ledger why <path>:<line> --trace` prints it ([CLI](cli.md)), in the same order and words: the session (its `s_…`, the agent, its turns and calls, and whether the model's thinking was exposed), then the turn (`Turn 1 · <when> · completed`) with its prompt, the model's thinking, its answers and narration, and each tool call with its input, its output, what it read and the lines it wrote (`wrote: inventory.py 1-3 (ai, …)`). A field this machine did not keep says why, in the words of the table above. A line no session on this machine wrote has no trace; the panel says so, as the CLI does.

**Full / Exchanges** switches the view (`--view exchanges`): Exchanges keeps the prompts, messages, edits, failed calls, questions, decisions and subagents, and folds every other run of calls and thinking into one row, `… 3 calls (Read 2, Grep 1), 1 thinking`. The panel keeps your choice for the next trace. What the agent said just before a call (its narration) reads **narration (Edit call):** when that call follows it on the page.

When the repository syncs and you are signed in, the hover and the panel also offer **Open the session's trace on the web**: the session's trace in the web app, from the link `lothal-ledger status --summary --json` gives as `web.trace`. It opens in your browser; the extension sends nothing there itself.

## Settings

| Setting | Default | |
|---|---|---|
| `lothalLedger.cliPath` | empty | The `lothal-ledger` program's path; empty: the one on `PATH`. |
| `lothalLedger.decorations.enabled` | on | The line marks (also **Lothal Ledger: Toggle Decorations**). |
| `lothalLedger.decorations.showUnknown` | on | Mark lines this machine has no record for. |
| `lothalLedger.decorations.showUnattributed` | on | Mark lines with no observed agent or tool write. |
| `lothalLedger.decorations.overviewRuler` | on | AI, changed-since and tool lines in the scroll bar. |
| `lothalLedger.codeLens.enabled` | on | The dossier line above each symbol. |
| `lothalLedger.hover.enabled` | on | The why card. |
| `lothalLedger.debounceMs` | 400 | How long after your last edit it asks again. |

## Troubleshooting

- **`Ledger: CLI not found`**: VS Code found no `lothal-ledger`. Install it, or set `lothalLedger.cliPath`; the menu copies the install command.
- **`Ledger: update the CLI`**: the `lothal-ledger` it found is older than 0.4.3. Run `lothal-ledger-update`, then **Lothal Ledger: Refresh**.
- **`Ledger: not enabled`**: nothing is recorded in this repository until you run `lothal-ledger enable` in it ([Enable and modes](enable-and-modes.md)); lines committed before then read No record.
- **No status bar item and no marks**: the folder is not trusted, or the file is not in a git repository.
- **Every line reads No record**: the commits were made before you enabled the repository, or on another machine. Signed in, the hover asks the team platform.

Next: [Zed](zed.md).
