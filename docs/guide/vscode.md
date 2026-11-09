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

The status bar then says the repository's mode, as `status` does: `Ledger: local`, `Ledger: local · content none` (the ignore rule `**` excludes the whole repository: a rule, not a mode, see [Keep content out](enable-and-modes.md#keep-content-out)), `Ledger: sync · not signed in`, or `Ledger: sync` while you are signed in, with what is waiting to upload (`Ledger: sync · 7 held · 1 failed`). Click it for the actions: open the status panel, run `doctor`, copy the sign-in command, refresh, turn the line marks off. A repository where you never ran `lothal-ledger enable` reads `Ledger: not enabled`, and its menu copies `lothal-ledger enable`, `lothal-ledger enable --sync`, or the ignore rule that keeps prompts and context out (attribution only): `mkdir -p .lothal-ledger && echo '**' >> .lothal-ledger/ignore`. The tooltip and the status panel's **This repository** card name the ignore files in force as `status` does, `content kept out by ignore rules: 1 rule in <repository>/.lothal-ledger/ignore`, and the card says which rule makes it `content none`.

When the store nears its storage cap, the item adds how full the store is, with a warning sign. Recording stops before the cap, at the capture allowance: the item then reads `not recording: budget` (or `store full` until the stop is marked), and its tooltip and the status panel's **Local store** card say it in the CLI's words, with the time it stopped: `not recording: budget: the store reached its capture allowance, so nothing new is recorded or attributed until there is room (raise storage.cap or delete sessions; lothal-ledger doctor)`, then `; since <time>` ([Storage](storage.md#when-the-store-is-full)). Once there is room, recording resumes, and both say so as `doctor` does: `recording: resumed at <time>; nothing was recorded from <time> to then (budget), and that gap is not filled`. The card also lists each tally of the store's gap counter, as `gap expired_unsent: 3 items, 9.3 KiB not kept`.

## What it shows

- **Line marks** in the gutter, one per line, in the line's state (below). AI, changed-since and tool lines are marked in the scroll bar too.
- **Hover** on a line: its *why* card. The state and the agent and model; the prompt of the turn that wrote the line (*Asked: …*), or why there is none; the session (`s_…`), the turn's number (`turn 1`), the writing session's steps in one line (`3 steps · 1 failed`) and the commit; the decisions recorded about it, each with the turn it was made in; links to the panels, with **Trace** on a line a session on this machine wrote ([below](#trace-the-turn-that-wrote-a-line)). In a file with unsaved changes, the card says to save it for the full story.
- **CodeLens** above each function and class: a one-line dossier of the symbol, for example `ai 2 · ai~ 1 · claude-code · claude-opus-5-5 · 1 decision` (two AI lines, one changed since, the agent and model, one decision).
- **Panels**, from the hover's links, the CodeLens, the editor's context menu (**Why This Line?**, **Show Trace of the Writing Turn**, **Show Dossier**) or the command palette (category *Lothal Ledger*): **Line** (the full story of a line), **Trace** (the turn that wrote it, below), **Dossier** (a symbol's card, its writes, decisions and history across renames), **Chain** (**Show Chain Across Teammates**: the symbol's history across sessions and, signed in, your teammates' sessions on the team platform) and **Status** (what `status` and `doctor` say).
- **The Lothal Ledger view** in the activity bar: **Sessions** recorded on this machine, each with its last activity, its title and model, its turns and its `s_…` (the first page `lothal-ledger sessions --json` answers). Click one to open its trace in the **Trace** panel from its first turn, as `lothal-ledger trace s_…` prints it; **Copy Session Reference** copies its `s_…`. In **Decisions in This File**, click a decision to open the call that recorded it, or the turn it was made in, in the Trace panel, as its row in the **Line** panel does; one with neither opens the **Dossier** of the function or class it names, else the **Line** panel, which lists every decision that names the file.

### The line states

| State | Mark | Hover says |
|---|---|---|
| AI ● | solid bar | An agent wrote this line. |
| AI, changed since ◐ | striped bar | An agent wrote this line, and a later change kept part of it. |
| Tool · *name* ▲ | solid bar | A tool (*name*) an agent ran rewrote this line. |
| Unattributed ○ | hairline | No agent or tool write is recorded for this line. |
| No record · | dashed outline | This machine has no record for this line. |

No record (`unknown`) means this machine has no record of the commit the line came from; it never means a person wrote it, and it is never shown as unattributed. Colour is never the only sign: each state has its own mark shape, and its own label in the hover. `lothalLedger.decorations.showUnknown` and `showUnattributed` hide those two marks.

### When the prompt is not shown

| The card says | Why |
|---|---|
| Not recorded: an ignore rule excludes it. | an ignore rule kept it out ([Keep content out](enable-and-modes.md#keep-content-out)): a per-path rule leaves out what names a file it matches, the whole-repository rule (`**`, what `content none` sets) leaves out all of it. The card says it in the CLI's words, the `content_message` of `lothal-ledger why` |
| Not kept: a capture budget ran out (the store was full, or redaction ran out of time). | it was recorded before 0.5.0 while your store was full ([Storage](storage.md)), or redaction ran out of time before it uploaded |
| Not kept: too large to keep. | it was over the item cap (`storage.item_cap`, 8 MiB by default); its hash and size are kept |
| Kept on the team store; removed from this machine. | uploaded, then removed here to make room |
| Lost: the agent's transcript changed. | recorded before 0.5.0: it was waiting in the agent's transcript, which changed before it was stored |
| Lost: the agent's transcript is gone. | the same, and the transcript was deleted |
| No prompt was tied to this call. | the agent wrote without a prompt the ledger could tie to it |

A line committed a moment ago can read *Still folding: the ledger is reading this commit. Try again in a moment.* while the ledger works out the commit in the background.
The extension asks the CLI to wait up to 1.5 seconds for a pending fold. If the
answer is still pending, the CLI supplies its cache lifetime; the next hover or
editor refresh after that lifetime asks again. There is no background retry loop.

A line committed while recording is stopped at the store's capture allowance (`not recording: budget`, [Storage](storage.md#when-the-store-is-full)) is not folded while the stop lasts, and reads *Not recorded: recording was stopped (not recording: budget); this commit was not folded.* in the hover, the gutter's hover and the **Line** panel, as `lothal-ledger why` says it, never *Still folding*.

A line of a commit only the store set aside at an upgrade folded (the legacy store, [Storage](storage.md)) reads *legacy store: the store set aside at the upgrade (read only, migrated in a later release, in Phase 8) answers this line; this machine's store has no record of its commit*, in the hover, the gutter's hover and the **Line** panel, and each session that store names carries the **legacy store** chip; the line's story is shown from there, never as *Committed, but not folded on this machine.*

### The writing turn and its steps

The **Line** panel's **Written by** card gives the turn that wrote the line by its number in the session's trace, `Turn 1 (completed)`, with the route `lothal-ledger trace s_… --turn 1` and **Open turn 1**, which opens that turn in the Trace panel with the writing call marked. A turn this machine did not record (the team platform's) shows its number alone.

Its **Steps** card lists the writing session's tool calls in the order they occurred, as `lothal-ledger why` prints them before any commit: `steps of session s_…`, the writing turn (`writing turn: turn 1, completed, commit …`, or `not committed`) and its prompt, then each call with its tool and kind, its outcome and exit code (`Bash (command) failed, exit 127`), the first line of its error, and **wrote this line** or **wrote the text this line displaced** on the calls that wrote the line or the text it replaced. A failed call is marked, and the card counts them (`3 steps · 1 failed`); *… more steps follow* when the session holds more than the answer lists.

### Decisions and who delegated the writer

The **Line** and **Dossier** panels list the decisions about the code, as `lothal-ledger why` prints them ([CLI](cli.md)):

- A decision the agent recorded with `record_decision` says where: **recorded in** the agent, the session, the turn and the call (`recorded in claude-code session s_fdb0f41c0176dc, turn 1, call toolu_01A`), then its route, `lothal-ledger trace s_… --turn 1`. **Open the call in its trace** opens that turn in the Trace panel with the call marked and scrolled into view; a call the page does not hold says so.
- A question you answered, a plan you reviewed or a permission you decided ends with the turn it was made in, `turn index: 2`, its route and **Open turn 2**; a row from the team platform gives the number alone.
- A question the agent asked you is a **question** row with its outcome (*answered*, *unanswered*, *cancelled*) and the session it was asked in (`asked in s_…`). Each question reads as the CLI writes it: `Q: Which floor should low_stock default to?  selected: 5  · also offered: 3 | per item`, or `typed answer` when you typed one (your words follow as *typed answer*), or `no answer`. Questions are on this machine only; the team platform's decisions carry none.
- A decision whose text an ignore rule excluded reads *Not recorded: an ignore rule excludes it.* where its summary would be, as `lothal-ledger why` prints it: in the **Line**, **Dossier** and **Chain** panels, the hover and **Decisions in This File**.

When the session that wrote the line was a subagent, its prompt is labelled **prompt (subagent)** (its parent's instruction, not yours), and each session that delegated it follows, nearest first: **delegated by** `s_…` (the agent and its agent type), the call that spawned it, and the prompt of that call's turn; or *no spawning call recorded*. A session forked, handed off or continued from another lists that session as **forked from**, **handed off from** or **continued from** `s_…`, where it branched, and the prompt of the turn it branched from.

## Trace: the turn that wrote a line

The hover's **Trace** link, **Show Trace of the Writing Turn** in the editor's context menu, or the panel's **Trace** tab opens the **trace** of the turn that wrote the line, as `lothal-ledger why <path>:<line> --trace` prints it ([CLI](cli.md)), in the same order and words: the session (its `s_…`, the agent, its turns and calls, and whether the model's thinking was exposed), then the turn (`Turn 1 · <when> · completed`) with its prompt, the model's thinking, its answers and narration, and each tool call with its input, its output, what it read and the lines it wrote (`wrote: inventory.py 1-3 (ai, …)`). A field this machine did not keep says why, in the words of the table above; a repository that records attribution only adds *This repository records attribution only (content none).* under the session, as the CLI does. A line no session on this machine wrote has no trace; the panel says so, as the CLI does. A page the legacy store holds says *From the legacy store: the store set aside at the upgrade (read only, migrated in a later release, in Phase 8); this machine's store has no record of this session.*, one from the team platform says *From the team platform: this machine's store has no record of this session.*, and one this machine's store answers with the team platform says *From this machine and the team platform: the fields this machine removed after uploading them are read from the team platform.*

**Full / Exchanges** switches the view (`--view exchanges`): Exchanges keeps the prompts, messages, edits, failed calls, questions, decisions and subagents, and folds every other run of calls and thinking into one row, `… 3 calls (Read 2, Grep 1), 1 thinking`. The panel keeps your choice for the next trace. What the agent said just before a call (its narration) reads **narration (Edit call):** when that call follows it on the page.

When the repository syncs and you are signed in, the hover and the panel also offer **Open the session's trace on the web**: the session's trace in the web app, from the link `lothal-ledger status --summary --json` gives as `web.trace`. It opens in your browser; the extension sends nothing there itself.

## Settings

| Setting | Default | |
|---|---|---|
| `lothalLedger.cliPath` | empty | The `lothal-ledger` program's path; empty: the one on `PATH`. |
| `lothalLedger.decorations.enabled` | on | The line marks (also **Lothal Ledger: Toggle Decorations**). |
| `lothalLedger.decorations.showUnknown` | on | Mark lines this machine has no record for. |
| `lothalLedger.decorations.showUnattributed` | on | Mark lines with no recorded agent or tool write. |
| `lothalLedger.decorations.overviewRuler` | on | AI, changed-since and tool lines in the scroll bar. |
| `lothalLedger.codeLens.enabled` | on | The dossier line above each symbol. |
| `lothalLedger.hover.enabled` | on | The why card. |
| `lothalLedger.debounceMs` | 400 | How long after your last edit it asks again. |

## Troubleshooting

- **`Ledger: CLI not found`**: VS Code found no `lothal-ledger`. Install it, or set `lothalLedger.cliPath`; the menu copies the install command.
- **`Ledger: update the CLI`**: the `lothal-ledger` it found is older than 0.4.3. Run `lothal-ledger-update`, then **Lothal Ledger: Refresh**.
- **`Ledger: not enabled`**: nothing is recorded in this repository until you run `lothal-ledger enable` in it ([Enable and modes](enable-and-modes.md)); lines committed before then read No record. `enable --content none` was removed in 0.5.0 and is refused: to record attribution only, the menu and the status panel copy the ignore rule `mkdir -p .lothal-ledger && echo '**' >> .lothal-ledger/ignore` instead.
- **`not recording: budget`**: the store reached its capture allowance, so nothing new is recorded or attributed until there is room, and a line an agent writes meanwhile reads Unattributed. Run `lothal-ledger doctor` for the remedy ([Storage](storage.md#when-the-store-is-full)). After `recording: resumed at …`, the lines written in the gap stay Unattributed: the gap is named, never filled.
- **`source changed` or `missing` in the Doctor card's Round trip row**: the row counts each agent's records as `doctor` does (`identical`, `identical apart from declared omissions`, `source changed`, `missing`); the agent rewrote or deleted its own file after the ledger read it. The ledger keeps what it read, and the check stays ok. Nothing to do ([The round trip](troubleshooting.md#the-round-trip)).
- **No status bar item and no marks**: the folder is not trusted, or the file is not in a git repository.
- **Every line reads No record**: the commits were made before you enabled the repository, or on another machine. Signed in, the hover asks the team platform.

Next: [Zed](zed.md).
