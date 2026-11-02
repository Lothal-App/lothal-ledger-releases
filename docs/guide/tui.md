# The terminal UI

`lothal-ledger-tui` shows what the ledger recorded, full screen in your terminal: this machine and repository at a glance, every line of a file with what wrote it, the whole story of one line, the sessions, search, and the upload of what you recorded before signing in. It reads the ledger by running `lothal-ledger … --json` in the repository you start it in, and changes nothing except the one upload you confirm on its Backfill tab.

## Install

The installer put `lothal-ledger-tui` in `~/.lothal-ledger/bin` beside `lothal-ledger` ([Install](install.md)), and `lothal-ledger-update` keeps the two at one version:

```console
$ lothal-ledger-tui --version
lothal-ledger-tui …
```

It runs the `lothal-ledger` on your `PATH`, which must be 0.4.3 or later (`--cli <path>` names another one).

## First run

The screens below are of the repository the earlier pages set up, after one more session whose work is committed (a line needs a commit to have a story). Start your agent in the repository and ask it `add a retry budget constant, RETRY_BUDGET = 3, in a new file retry.py`; when it has written the file, end the session and commit the file:

```console
$ claude
Lothal Ledger: recording this session in ~/src/your-project (sync, not signed in: kept on this machine)
$ git add retry.py
$ git commit -qm "Add a retry budget"
```

Start the terminal UI inside the repository (or anywhere with `-C <directory>`). It opens on the **Overview**: this repository's mode and content setting, whether you are signed in and what waits to upload, how full the store is, what `doctor` says and the agents it found. `j` and `k` scroll down to the doctor and agent boxes on a small terminal.

```console
$ lothal-ledger-tui
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 ╭ THIS REPOSITORY ───────────────────────────────────────────────────────────────────────────────╮
 │ path         ~/src/your-project…│
 │ recording    ● yes                                                                             │
 │ mode         sync  uploads while signed in                                                     │
 │ content      full  prompts and context are kept                                                │
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

The doctor and agent boxes report different things. `Every check passes.` means the setup checks passed; the agents box can still count warnings, including agents that are not installed or whose version could not be determined. In v0.4.5 a healthy setup can therefore show 13 agent warnings alongside the doctor's OK status. Press `a` to expand the agent details and read the warnings; the warning count alone does not mean capture is broken.

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

The bottom line is the cursor's line: its state, agent, model when known, commit and session. `S` opens that session, `s` the function or class around the line, `t` the trace of the turn that wrote it ([below](#trace-the-turn-that-wrote-a-line)).

## Why: the whole story of a line

`⏎` on a line opens **Why**: the commit, the session and the call that wrote the line, the turn and its prompt, what the model saw, the decisions recorded about it, and every session behind the commit. `h` and `l` fold and unfold, `⏎` on a session opens it.

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
         ▾ turn 4acc7782-8605-47b2-a674-…  completed
             prompt add a retry budget constant, RETRY_BUDGET = 3, in a new file retry.py
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

### When the prompt is not shown

A turn shows its prompt when this machine kept it. Otherwise the line in its place says why:

| What it says | Why |
|---|---|
| Not recorded: this repository records attribution only. | the repository is `content none` |
| Not kept: a capture budget ran out (the store was full, or redaction ran out of time). | the store was full ([Storage](storage.md)), or redaction ran out of time |
| Kept on the team store; removed from this machine. | uploaded, then removed here to make room; signed in, it is read from the team platform |
| In the agent's transcript, not stored yet. | the store had no room; it is stored once there is |
| Lost: the agent's transcript changed. | it was waiting in the agent's transcript, which changed before it was stored |
| Lost: the agent's transcript is gone. | the same, and the transcript was deleted |
| No prompt was tied to this call. | the agent wrote without a prompt the ledger could tie to it |
| No prompt is retained for this turn on this machine. | any other reason this machine holds no text for the turn |

`What the model saw` is the context the model was given for the call; for agents whose requests are not captured it says `What the model saw was not captured for this agent.`

## Trace: the turn that wrote a line

`t` on a line in Blame, or in a line's Why, opens the **trace** of the turn that wrote it, as `lothal-ledger why <path>:<line> --trace` prints it ([CLI](cli.md)): the session, then the turn's prompt, the model's thinking, its answers and narration, and each tool call with its input, its output, what it read and the lines it wrote. A field this machine did not keep says why, in the words of the table above. `j` and `k` scroll, `esc` goes back to the why.

```console
$ lothal-ledger-tui retry.py:1  # then t
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 Trace  › retry.py:1  the turn that wrote it

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
 j/k scroll  esc back to why  r refresh  / search  ? help  q quit
```

When the repository syncs and you are signed in, the trace also shows `Session trace on the web:` and the session's trace in the web app (the link `lothal-ledger status --summary --json` gives as `web.trace`); `y` copies it.

## Sessions, search and backfill

**Sessions** (`4`) lists the sessions recorded on this machine, newest first; `a` and `s` filter by agent and state, `⏎` opens a session's card (its reference, agent, turns and events):

```console
$ lothal-ledger-tui --tab sessions
 ▄▟▙▄ Lothal │ Ledger                                          ~/src/your-project   ● sync   v…
  1 Overview   2 Blame   3 Why   4 Sessions   5 Search   6 Backfill
 Filter › f   agent all a   state all s…sessions

   LAST SEEN        AGENT         STATE   TURNS EVENTS SESSION
 › … claude-code   ended   1     …s_…
   … claude-code   ended   1     …s_…
   … claude-code   ended   1     …s_…
…
 j/k move  ⏎ card  f filter  a agent  s state  x clear  r refresh  / search  ? help  q quit
```

**Search** (`5` or `/` anywhere) searches the prompts and responses kept on this machine and, signed in, the team platform's. **Backfill** (`6`) previews what `lothal-ledger sync --backfill-local` would upload ([Sign in and sync](login-and-sync.md#upload-what-you-recorded-before-signing-in)); `c` includes the content, `u` uploads after asking you to confirm.

`?` lists every key; `r` reads everything again; `q` quits and gives the terminal back as it was.

## Troubleshooting

- **lothal-ledger is not installed or not on PATH**: the terminal UI found no `lothal-ledger` to run. Install it ([Install](install.md)), or start it with `--cli <path>`.
- **`is too old for this UI`**: the `lothal-ledger` it found is older than 0.4.3. Run `lothal-ledger-update`, then press `r`.
- **`No answer within 15 s`**: a `lothal-ledger` call took too long and was stopped so the screen stays responsive. A large store can be slow: start it with `--timeout <seconds>`.
- **`not a git repository`**: Blame and Why read a repository's files. Start it inside one, or pass `-C <directory>`.
- **No colours, or the wrong ones**: `--color never` (or `NO_COLOR=1`), `256` or `truecolor`; every state keeps its glyph and bar without colour.

Next: [VS Code](vscode.md).
