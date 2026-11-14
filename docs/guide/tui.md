# The terminal UI

`lothal-ledger-tui` shows what the ledger recorded, full screen in your terminal: this machine and repository at a glance, every line of a file with what wrote it, the whole story of one line, the decisions stated about a line or a function, the sessions, search, and the upload of what you recorded before signing in. It reads the ledger by running `lothal-ledger … --json` in the repository you start it in, and changes nothing except the one upload you confirm on its Backfill tab.

## Install

The installer put `lothal-ledger-tui` in `~/.lothal-ledger/bin` beside `lothal-ledger` ([Install](install.md)), and `lothal-ledger-update` keeps the two at one version:

```console
$ lothal-ledger-tui --version
lothal-ledger-tui …
```

It runs the `lothal-ledger` on your `PATH` (`--cli <path>` names another one) and reads its version first (`--version --json`); one too old for this UI says so ([Troubleshooting](#troubleshooting)).

## First run

The screens below are of the repository the earlier pages set up, after one more session whose work is committed (before the commit, Why reads a line from the working log, `Uncommitted: read from the working log`, and has no commit to show). Start your agent in the repository and ask it `add a retry budget constant, RETRY_BUDGET = 3, in a new file retry.py`; when it has written the file, end the session and commit the file:

```console
$ claude
Lothal Ledger: recording this session in ~/src/your-project (sync, not signed in: kept on this machine)
$ git add retry.py
$ git commit -qm "Add a retry budget"
```

Start the terminal UI inside the repository (or anywhere with `-C <directory>`). It opens on a quick **Overview** from `status --summary --json`: this repository's mode and content setting, whether you are signed in, what waits to upload, and the storage state. Press `d` to load the full repository list, sync and storage details, and doctor checks; `a` loads those details and opens the agents. `j` and `k` scroll down on a small terminal.

```console
$ lothal-ledger-tui  # then d
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 ╭ THIS REPOSITORY ───────────────────────────────────────────────────────────────────────────────╮
 │ path         ~/src/your-project…│
 │ recording    ● yes                                                                             │
 │ mode         sync  uploads while signed in                                                     │
 │ content      full                                                                              │
 │              recorded except what ignore rules exclude                                         │
 ╰────────────────────────────────────────────────────────────────────────────────────────────────╯
 ╭ REPOSITORIES · 1 ENABLED ──────────────────────────────────────────────────────────────────────╮
 │ ● ~/src/your-project         sync…│
 ╰────────────────────────────────────────────────────────────────────────────────────────────────╯
 ╭ SYNC ──────────────────────────────────────────────────────────────────────────────────────────╮
 │ account      ○ not signed in                                                                   │
 │              Nothing leaves this machine until you sign in.                                    │
 │ sign in       lothal-ledger login                                                              │
 │ paused       logged out                                                                        │
 │ queue        0 pending  … held  0 dead  0 withheld…│
…
 │ last upload  …│
 │ flusher      …│
 ╰────────────────────────────────────────────────────────────────────────────────────────────────╯
 ╭ STORAGE ───────────────────────────────────────────────────────────────────────────────────────╮
 │ ░…│
 │ … of the 4.0 GiB cap  cap from user…│
 │ ● Healthy: warns at 80 %.                                                                      │
 │ free space   …│
 ╰────────────────────────────────────────────────────────────────────────────────────────────────╯
…
 y copy command  j/k scroll  a agents  r refresh  / search  ? help  q quit
```

`content` says what the ignore rules keep out ([Privacy](privacy.md)): `full`, `recorded except what ignore rules exclude`, with how many rules in how many ignore files are in force, or, when a `**` rule excludes the whole repository, `none`, `an ignore rule (**) excludes the whole repository: hashes and sizes only`. To keep content out, add rules to `.lothal-ledger/ignore`; there is no content mode to turn off.

STORAGE measures everything the ledger keeps on this machine, each repository's `.git/lothal` included, against the cap ([Storage](storage.md)). When the store reaches its capture allowance, 95 % of the cap, recording stops: the header says `○ not recording: budget`, the `recording` row turns to `○ no`, and STORAGE says, in the words `lothal-ledger status` uses, that nothing new is recorded or attributed until there is room, since when, and what frees room. Recording also stops when free space on the disk falls below `storage.reserve`; STORAGE then says so in that stop's words: `not recording: budget: free space on the store's filesystem is below storage.reserve, so nothing new is recorded or attributed until there is room (free disk space or lower storage.reserve; lothal-ledger doctor)`. Recording resumes by itself once there is room again. A line committed while recording is stopped is not folded: Why says `Not recorded: recording was stopped (not recording: budget); this commit was not folded.`, and Blame does not promise a fold that will not come.

The doctor and agent boxes report different things. `Every check passes.` means the setup checks passed; the agents box can still count warnings, for example for an installed agent whose version could not be determined; an agent that is not installed carries none. Press `a` to expand the agent details and read the warnings; the warning count alone does not mean capture is broken. The doctor box also shows the round trip, as `lothal-ledger doctor` prints it (`Round trip: ok; … records (… bytes) checked in … streams`), with each record whose source changed or went missing; then it does not say `Every check passes.`

After `lothal-ledger delete-session` of a session the team store holds, the SYNC box lists it, on the first screen and after `d` alike, as `deleted … · team delete: pending (Phase 7)`: the team copy stays until Phase 7.

Signed out, nothing uploads: what a `sync` repository records meanwhile stays on this machine and never uploads by itself ([Sign in and sync](login-and-sync.md#upload-what-you-recorded-before-signing-in)). What was already waiting to upload when you signed out is counted as `held`, with the line `held (logged_out)` under the queue, until you sign in again as the same account; it is `0 held` when everything had uploaded first. Where a box says what to run (here `lothal-ledger login`), `y` copies the command; the terminal UI never signs in, enables or uploads by itself.

## Blame: what wrote each line

Open a file in **Blame** by naming it (`<file>:<line>` puts the cursor on that line), or press `2` and pick one. Each line has its state, the agent, and a bar in the gutter whose shape carries the state without colour:

```console
$ lothal-ledger-tui retry.py
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 retry.py  worktree  1 line                      ● ai  ◐ ai~  ▲ tool  ○ unattributed  · no record
     STATE AGENT         CODE
 ▌ 1 ai    claude-code  │ RETRY_BUDGET = 3
…
 ──────────────────────────────────────────────────────────────────────────────────────────────────
 L1  ● AI  claude-code  claude-opus-5-5  …  s_…
 j/k move  ⏎ why  s symbol  t trace  S session  o open file  r refresh  / search  ? help  q quit
```

On a wider terminal, Blame also shows a `MODEL` column; narrower windows hide that column first, then the agent column. These captures use a 100-column terminal, so the model appears in the bottom line rather than in a separate column.

The states are the same everywhere the ledger shows a line, and there is no state for a person:

| State | Shown as | Means |
|---|---|---|
| AI | `ai`, ● and a solid bar `▌` | an agent wrote the line |
| AI, changed since | `ai~`, ◐ and a striped bar `▞` | an agent wrote it, and a later change kept part of it |
| Tool · *name* | the tool's name, ▲ and a solid bar | a tool an agent ran (a formatter, a codemod) rewrote it |
| Unattributed | blank, ○ and a hairline `│` | no agent or tool write was observed for it |
| No record | `?`, · and a dashed bar `┆` | this machine has no record of the commit the line came from (`unknown`): never read it as unattributed |

The bottom line is the cursor's line: its state, agent, model when known, commit and session. `S` opens that session, `s` the cached dossier card of the function or class around the line, `d` the decisions stated about the line ([below](#decisions-what-was-stated)), and `t` the trace of the turn that wrote it ([below](#trace-the-turn-that-wrote-a-line)). The file's cards are read together once with Blame. The card is the compact L0 summary, not a partial full story: in Why, press `o` and enter `path:name` to read the symbol's complete writes, decisions and lineage. Enter on a Blame line still opens its line story, including the CLI's enclosing-symbol card when supplied.

## Why: the whole story of a line

`⏎` on a line opens **Why**: the commit, the session and the call that wrote the line, the turn and its prompt, the steps of the writing session, what the model saw, the decisions recorded about it, and every session behind the commit. `h` and `l` fold and unfold, `⏎` on a session opens it, and `⏎` on a turn opens that turn's trace at the call.

```console
$ lothal-ledger-tui retry.py:1  # then ⏎
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 Why  › retry.py:1

 ▾ retry.py:1  ● AI
     commit …  Add a retry budget
     model claude-opus-5-5
   ▾ wrote s_…::t_…
     ▾ claude-code session …  ended
       ▾ call PostToolUse Write  claude-opus-5-5
         ▾ turn 1  completed
             prompt add a retry budget constant, RETRY_BUDGET = 3, in a new file retry.py
   ▸ Steps of session s_…  1 step
   ▸ What the model saw  What the model saw was not captured for this agent.
     Decisions  none recorded
   ▸ Commit sessions (1)
…
 ╭ DETAILS ───────────────────────────────────────────────────────────────────────────────────────╮
 │ state    AI                                                                                    │
 │ commit   …│
 │ summary  Add a retry budget                                                                    │
 │ source   local                                                                                 │
 │ key      s_…::t_…│
 │ model    claude-opus-5-5                                                                       │
 ╰────────────────────────────────────────────────────────────────────────────────────────────────╯
 j/k move  h/l fold  ⏎ open  c chain  t trace  o ask  b blame  r refresh  ? help  q quit
```

`c` asks for the line's chain across teammates (signed in, the team platform's sessions too); `t` opens the trace of the turn that wrote the line; `o` asks about another line or a symbol (`path:line` or `path:name`).

### History: every write behind the line

`H` in a line's Why shows the line's **history** in place of the tree, as `lothal-ledger why <path>:<line>` lists it under `history (newest first)` ([CLI](cli.md#why-a-line-exists-why)): one row per write behind the line, newest first, back to the oldest one the ledger recorded. Each row gives the writing session's agent, the model, the turn number, when it wrote, its state (`● AI`, or `▲ Tool · ruff` for a formatter an agent ran) and the first line of the turn's prompt, or the line that says why the prompt is not shown ([below](#when-the-prompt-is-not-shown)). When the oldest write replaced a line nobody recorded, the last row reads `○ Unattributed: a write the ledger did not record`; when older writes are recorded than the answer holds, a line under the rows says so. The details under the rows name the selected write, its session and its `route`, the `lothal-ledger trace s_… --turn N` that opens its turn.

```console
$ lothal-ledger-tui retry.py:1  # then ⏎ H
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 History  › retry.py:1

   claude-code  claude-opus-5-5  turn 1  …  ● AI  add a retry budget constant, …

 ╭ DETAILS …╮
 │ write         s_…::t_…│
 │ session       s_…│
 │ conversation  …│
 │ session hash  …│
 │ route         lothal-ledger trace s_… --turn 1…│
 │…│
 │ add a retry budget constant, RETRY_BUDGET = 3, in a new file retry.py…│
 │…│
 │ ⏎ open this turn's trace…│
…
 j/k move  ⏎ open its turn  esc back to why  r refresh  / search  ? help  q quit
```

`j` and `k` move from write to write, `⏎` opens the trace of the selected write's turn ([below](#trace-the-turn-that-wrote-a-line)), `esc` there comes back to the history, and `esc` in the history goes back to the Why tree. `r` asks again and keeps the history on screen. `H` is not listed in the Why view's footer or its help.

### Subagents, questions and where a decision was recorded

The Why reads as `lothal-ledger why` prints it ([CLI](cli.md)):

- A session another agent spawned is marked `subagent`, and its own prompt reads `prompt (subagent)`: it is the parent agent's instruction, not a person's. Under the session, each session that delegated it follows, nearest parent first: `delegated by s_… (omp task) call … at …` and, under it, the turn and prompt of the call that spawned it, or `delegated by …: no spawning call recorded`.
- A question the agent asked (its question tool) reads `question answered: ask` (or `not answered`, `cancelled`) with `asked in s_…`, the session it was asked in. Each of its questions is a `Q:` line: what was picked and what else was offered (`selected: Hi  · also offered: Hello | Hey`), or `typed answer`, whose words are in the details.
- A decision the agent recorded with `record_decision` has a `recorded in` line: the session (its name when it has one), the turn and the call. `⏎` on it opens that turn's trace with the call at the top.
- A decision linked to the one session active when it was recorded reads `linked by time window to s_…`, and its details name that session, in the words `lothal-ledger why` uses for it.
- A decision whose text an ignore rule excluded reads `Not recorded: an ignore rule excludes it.` in place of its summary, never a blank row.

A turn is named by its number in the session's trace, `turn 1`, the number `lothal-ledger trace s_… --turn 1` takes; the details show that command as `route`, and the turn's own id as `turn id`. A question, plan or permission row made in a known turn has a `turn N  lothal-ledger trace s_… --turn N` line under it; `⏎` on it opens that turn's trace.

### The steps of the writing session

`Steps of session s_…` lists the writing session's tool calls in the order they ran, from the write on, as `lothal-ledger why` prints its `steps of session` list: first `writing turn:` with the turn's number, model, status and commit (`not committed` before one), then each call, `1. Write (edit) succeeded  [wrote this line]`, with its exit code, `test` for a test run, and the first line of its error. The call whose text the line displaced reads `[wrote the text this line displaced]`. The outcome is a word (`succeeded`, `failed`, `interrupted`, `denied`, `unknown`), so it reads the same without colour. `⏎` on a step opens its turn's trace at that call.

### When the prompt is not shown

A turn shows its prompt when this machine kept it. Otherwise the line in its place says why:

| What it says | Why |
|---|---|
| Not recorded: an ignore rule excludes it. | an ignore rule excludes the whole repository (`**`); in a trace, also a call that names a path an ignore rule excludes |
| Not kept: a capture budget ran out (the store was full, or redaction ran out of time). | redaction ran out of time when it was uploaded (read from the team platform); a full store on this machine stops recording instead ([Storage](storage.md)) |
| Not kept: too large to keep. | an item over `storage.item_cap` (8 MiB by default), such as a long tool output |
| Kept on the team store; removed from this machine. | uploaded, then removed here to make room; signed in, it is read from the team platform |
| Removed: past its retention, or deleted. | its retention period ended ([Storage](storage.md)), or it was deleted from the team store |
| No prompt was tied to this call. | the agent wrote without a prompt the ledger could tie to it |
| No prompt is retained for this turn on this machine. | any other reason this machine holds no text for the turn |

`What the model saw` is the context the model was given for the call; for agents whose requests are not captured it says `What the model saw was not captured for this agent.`

## Trace: the turn that wrote a line

`t` on a line in Blame, or in a line's Why, opens the **trace** of the turn that wrote it, as `lothal-ledger why <path>:<line> --trace` prints it ([CLI](cli.md)): the session, then the turn's prompt, the model's thinking, its answers and narration, and each tool call with its input, its output, what it read and the lines it wrote. A field this machine did not keep says why, in the words of the table above; a field read from the team platform can also say the lines [Privacy](privacy.md#what-you-see-instead-of-the-text) lists. A trace read from the team platform opens with `From the team platform: this machine's store has no record of this session.` `j` and `k` scroll, `esc` goes back to the why.

`v` switches to the **exchanges** view and back (`--view exchanges`): the prompts, the answers, the edits, the failed calls, the questions, the recorded decisions and the subagents stay, and every other run of calls and thinking folds into one `…` line that counts them by tool and names the command that prints the whole turn. The view stays for the next trace you open.

```console
$ lothal-ledger-tui retry.py:1  # then t
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 Trace  › retry.py:1  the turn that wrote it  view full

 Session s_… · claude-code · 1 turn · 1 call · thinking: text
 Models: claude-opus-5-5

 Turn 1 · … · completed · claude-opus-5-5
   prompt: add a retry budget constant, RETRY_BUDGET = 3, in a new file retry.py
   model: - → claude-opus-5-5
   thinking (text): Use the retry budget for the delay sequence.
   call Write […] · ok
…
     output: File created successfully at: ~/src/your-project/retry.py
…
 j/k scroll  v exchanges  esc back to why  r refresh  / search  ? help  q quit
```

When the repository syncs and you are signed in, the trace also shows `Session trace on the web:` and the session's trace in the web app (the link `lothal-ledger status --summary --json` gives as `web.trace`); `y` copies it.

## Decisions: what was stated

`d` on a line in Blame, or in a Why (for the line or the symbol it asked about), opens the **Decisions** view in its place: only what agents and people stated about that line or symbol, as `lothal-ledger decisions <path>:<line>` or `<path>:<symbol>` lists it ([CLI](cli.md#what-was-decided-decisions)), newest first. It is the second kind of information, labelled as stated and kept apart from the recorded history Why shows. The view opens with the CLI's sentence saying what the list is and where the history is, then each decision as the CLI words it: when and who stated it (`agent claude-code`, or `person` for a person), `decision` and its id, then its summary; `why:` and its rationale; `recorded in:` the agent, session, turn and call that recorded it, and the route to that turn, `lothal-ledger trace s_… --turn 1`. A decision from the team platform is marked `[team platform]`; one whose text an ignore rule excluded reads `Not recorded: an ignore rule excludes it.` in place of its summary and rationale; one with no recording call reads `recorded in: no recording call known`.

`j` and `k` move from decision to decision, `⏎` opens the trace of the turn that recorded the selected one, at its call, and `esc` there comes back to the Decisions view (`esc back to decisions`); `esc` in the view goes back to the Blame or Why it was opened from, as you left it. Where nothing names the line, the view says so and names the command that gives its recorded history:

```console
$ lothal-ledger-tui retry.py:1  # then d
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 Decisions  › retry.py:1
…
            ○ No stated decisions

            no decision stated by an agent or a person names retry.py:1; the recorded
            history: lothal-ledger why retry.py:1
…
 esc back to blame  r refresh  / search  ? help  q quit
```

## Sessions, search and backfill

**Sessions** (`4`) lists the sessions recorded on this machine, newest first, with the model each ran on where the terminal is wide enough; `a` and `s` filter by agent and state, `⏎` opens a session's card (its reference, agent, model, repository, turns, edits and events, and its title when it has one):

```console
$ lothal-ledger-tui --tab sessions
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 Filter › f   agent all a   state all s…sessions

   LAST SEEN        AGENT         MODEL            STATE   TURNS EVENTS SESSION
 › … claude-code   claude-opus-5-5  ended   1     …s_…
   … claude-code   claude-opus-5-5  ended   1     …s_…
   … claude-code   claude-opus-5-5  ended   1     …s_…
…
 j/k move  ⏎ card  f filter  a agent  s state  x clear  r refresh  / search  ? help  q quit
```

**Search** (`5` or `/` anywhere) searches the prompts and responses kept on this machine and, signed in, the team platform's. **Backfill** (`6`) previews what `lothal-ledger sync --backfill-local` would upload ([Sign in and sync](login-and-sync.md#upload-what-you-recorded-before-signing-in)); `c` includes the content, `u` uploads after asking you to confirm.

`?` lists every key; `r` reads everything again; `q` quits and gives the terminal back as it was.

## Troubleshooting

- **lothal-ledger is not installed or not on PATH**: the terminal UI found no `lothal-ledger` to run. Install it ([Install](install.md)), or start it with `--cli <path>`.
- **`is too old for this UI`**: the `lothal-ledger` it found is older than the terminal UI needs; the screen names the version it needs. Run `lothal-ledger-update`, which keeps the two at one version, then press `r`.
- **`No answer within 15 s`**: a `lothal-ledger` call took too long and was stopped so the screen stays responsive. A large store can be slow: start it with `--timeout <seconds>`.
- **`not a git repository`**: Blame and Why read a repository's files. Start it inside one, or pass `-C <directory>`.
- **No colours, or the wrong ones**: `--color never` (or `NO_COLOR=1`), `256` or `truecolor`; every state keeps its glyph and bar without colour.

Next: [VS Code](vscode.md).
