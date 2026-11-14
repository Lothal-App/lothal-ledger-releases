# Ask from the shell

Every question here is answered from this machine first; signed in, what this machine cannot answer comes from the team platform only for a repository enabled with `--sync` ([Sign in and sync](login-and-sync.md)). The file's or recorded session's repository decides, even when you ask from elsewhere; `sessions` and `search` ask about the current repository. A `local` or not-enabled repository sends nothing: the command says why and points to `lothal-ledger enable --sync` in that repository. `lothal-ledger <command> --help` gives each command's full reference.

The examples ask about one more piece of work, a function this time. Ask your agent `add a function delays(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of them`; when it has written the file, end the session and commit it:

```console
$ claude
Lothal Ledger: recording this session in ~/src/your-project (sync, not signed in: kept on this machine)
$ git add backoff.py
$ git commit -qm "Add retry delays" -m "Double each wait using the shared retry budget."
```

## Who wrote each line: `blame`

`lothal-ledger blame <file>` is `git blame` with each line's state and agent first, and takes every `git blame` option (`-L 10,20`, `-w`, …):

```console
$ lothal-ledger blame backoff.py
…
ai claude-code claude-opus-5-5  … 1) from retry import RETRY_BUDGET
ai claude-code claude-opus-5-5  … 2)
ai claude-code claude-opus-5-5  … 3)
ai claude-code claude-opus-5-5  … 4) def delays(base):
ai claude-code claude-opus-5-5  … 5)     return [base * 2**n for n in range(RETRY_BUDGET)]
```

| State | Means |
|---|---|
| `ai` | an agent session wrote the line |
| `changed_since` | an agent wrote it, and a later change nobody recorded kept part of it |
| `tool:<name>` | a tool an agent ran rewrote it (`rustfmt`, `prettier`, …) |
| `unattributed` | no agent or tool write was observed |
| `unknown` | this machine has no record of the commit; never read it as unattributed |

There is no state for a person. Right after a commit, the ledger may still be working out which lines it holds: `blame` then says `commit … is being remapped; waiting up to 2.0 s` and answers once it knows, within 2 seconds, or marks the lines `pending`. `--wait-ms <n>` sets that wait (0 to 5,000 milliseconds): an editor asks with a short one and asks again later.

## Why a line exists: `why`

`lothal-ledger why <file>:<line>` gives the line's state, the commit, every write behind the line, newest first, and the session, call and turn that wrote it, with your prompt:

```console
$ lothal-ledger why backoff.py:5
backoff.py:5  ai  (commit … Add retry delays; local)
        return [base * 2**n for n in range(RETRY_BUDGET)]
commit body:
    Double each wait using the shared retry budget.
model: claude-opus-5-5
history (newest first):
  s_… turn 1  claude-opus-5-5  …  ai
    prompt: add a function delays(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of th …
    lothal-ledger trace s_… --turn 1
wrote s_…::t_…
  claude-code session … (ended, …)
    call PostToolUse Write … at …
      thinking (text, 44 bytes): trace block …
    turn 4acc7782-8605-47b2-a674-… (event) started …
      turn index: 1
      lothal-ledger trace s_… --turn 1
      prompt: add a function delays(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of them
steps of session …:
  writing turn: turn 1, model claude-opus-5-5, completed, commit …
    prompt: add a function delays(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of them
  1. … Write (edit) succeeded  [wrote this line]
commit sessions:
  claude-code …
```

Under `history (newest first)` it lists every write behind the line, back to the oldest one the ledger recorded: here one, the agent's; after a formatter run and a later edit by another session, three, each with its session, turn number, model, date, state and prompt, and the `trace` command that opens its turn. When the oldest write replaced a line nobody recorded, the list ends with `unattributed`. Below it come the writer's call, turn and prompt in full, the write the line replaced, and other sessions linked to the commit. The decisions you or the agent stated are counted in one line, `decisions: 2 stated; lothal-ledger decisions backoff.py:5`, the command that lists them. Under `steps of session`, it lists the writing session's tool calls in the order they ran, recorded as the session ran, and marks the one that wrote the line. When the prompt is not shown, a line in its place says why ([Privacy](privacy.md#what-you-see-instead-of-the-text)). `--context` adds what the model had been given before it wrote the line.

The steps are there before any commit, as soon as the agent has written the line. Ask about a line an agent wrote, tested, rewrote and tested again, and the steps show that story: the first write, marked `[wrote the text this line displaced]`, the test command that failed with its exit code (`failed, exit 101` for a failing `cargo test`), the rewrite, marked `[wrote this line]`, and the test that passed. The writing turn says `not committed` until a commit holds the line, then names the commit; the steps stay the same. `why` lists at most 24 steps; when more follow, it says so and `trace` shows the rest.

`turn index: 1` and `lothal-ledger trace s_… --turn 1` under the writing turn open that turn of the session's trace. A decision made in a turn carries the same route: a question you answered, a plan you reviewed or a permission you decided ends with the two lines of the turn you decided it in, and a decision the agent recorded follows its `recorded in:` line with the `trace` command for its call's turn. A question you answered in turn 2 of a session that wrote the line in turn 5 routes to turn 2:

```text
decisions:
  … implicit (human) omp, session … (captured in it): question answered: ask (asked in s_…)
    Q: Which greeting should I add?  selected: Hi  · also offered: Hello | Hey
    turn index: 2
    lothal-ledger trace s_… --turn 2
```

Besides the decisions that name the file, `why` lists the ones that led to the line: a plan approved, a question answered or a permission refused in the writing session at or before its write, in any of its turns (and, for a subagent's line, in the session that started it, before it did), the permission for the call that wrote the line, and the decisions the agent recorded before the write. A decision made after the write is listed only when it names the file or the function.

The commit's prose appears under `commit body:` (at most 4,096 UTF-8 bytes). Git trailers such as `Lothal-Session`, `Co-authored-by` and `Signed-off-by` are not prose: a commit with only those after its subject has no body section. For a commit this machine never folded, session trailers appear separately as commit hints, never as proof of which session wrote the line.

A line of a commit only a store of another build folded reads `unknown`: the fresh store never folded its commit ([A store of another build](storage.md#a-store-of-another-build)). `trace` reads the current store only: a session only the set-aside store holds is unknown to it.

## Why a function exists: `why <file>:<function>` and `dossier`

Name a function, class or method instead of a line, and `why` follows it across renames and moves: its lines by state, the session that first wrote it, every write into it and each decision about it. `--all-users` adds the sessions and decisions of everyone on the team platform who touched it.

```console
$ lothal-ledger why backoff.py:delays
backoff.py:delays  function L4-5  key …
lines: ai 2, ai, changed since 0, unattributed 0
origin: claude-code claude-opus-5-5 … s_… (1 session with AI writes)
wrote s_…::t_…
…
      prompt: add a function delays(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of them
…
```

`dossier` gives the short card of every function and class in a file (or of one, with `<file>:<name>`): its lines, their states, the write that first made it and its newest decision. A file an ignore rule excludes has no card: `dossier` says the rule excludes it, and so do the `symbols` command and the agents' `dossier` tool, since a function's name is the file's content. The card an agent gets after reading it is not made either, and the team platform is never asked about its functions.

```console
$ lothal-ledger dossier backoff.py
function delays L4-5: ai 2, unattributed 0; origin claude-code claude-opus-5-5 … s_…
```

The agents get the same card when they read the file ([Inside the agent](in-agent.md#the-card-when-the-agent-reads-a-file)).

## What was decided: `decisions`

`why` gives the recorded history. `lothal-ledger decisions <file>`, `<file>:<line>` or `<file>:<function>` gives only what agents and people stated about that code, for a quick review: the decisions that name it, newest first, each with its date, who stated it, its summary and rationale, and the route to the turn that recorded it. They are stated, not recorded: a second kind of information, labelled as such and kept apart from the history `why` shows, since an agent may state a decision badly or not at all. A decision the agent derived (`ai_derived`) and what a session captured on its own (a question answered, a plan approved, a permission decided) are not listed here; `why` lists them. Signed in, a `sync` repository adds your teammates' stated decisions.

An agent states a decision when you ask it to, or when it chooses between real alternatives ([Inside the agent](in-agent.md#asking-the-ledger-from-the-agent)). Ask your agent `Record a decision with lothal-ledger: each retry waits twice as long as the one before, at most RETRY_BUDGET waits, because the upstream API rate-limits bursts; it applies to delays in backoff.py`. This is what the agent sends, a call of the ledger's `record_decision` tool with the decision's summary, its rationale and the code it concerns:

```json mcp=record_decision
{
  "summary": "Each retry waits twice as long as the one before, at most RETRY_BUDGET waits",
  "rationale": "The upstream API rate-limits bursts: a fixed wait retries too soon.",
  "paths": ["backoff.py"],
  "symbols": [{"path": "backoff.py", "name": "delays"}]
}
```

Then ask the ledger what was decided about the function:

```console
$ lothal-ledger decisions backoff.py:delays
decisions stated by agents and people about backoff.py:delays, newest first; the recorded history: lothal-ledger why backoff.py:delays
  …  agent claude-code  decision …: Each retry waits twice as long as the one before, at most RETRY_BUDGET waits
    why: The upstream API rate-limits bursts: a fixed wait retries too soon.
    recorded in: no recording call known
```

The first line says what the list is and where the recorded history is. Each decision then gives when it was stated, who stated it (`agent claude-code`, or `person` for a decision a person stated), its id and summary, `why:` and its rationale, and `recorded in:`, the agent, session, turn and call that recorded it, which the ledger reads from the session's transcript, with the route to that turn under it, `lothal-ledger trace s_… --turn 1`. A call no transcript holds, as in this example, reads `recorded in: no recording call known`. A decision from the team platform is marked `[team platform]`, and one whose text an ignore rule excludes reads `Not recorded: an ignore rule excludes it.` in place of its summary and rationale. With none stated, `decisions` prints its first line only, which says that no decision names the target. `lothal-ledger decision <id>` shows one decision with the paths and symbols it names as they are now.

## Sessions and search

`sessions` lists the recorded sessions, newest first: when, the agent, the model, the repository, turns and edits, and the `s_…` reference `why` prints. It shows 100 at a time (`--limit` sets how many) and ends a page with the command for the next:

```console
$ lothal-ledger sessions --limit 2
… claude-code  claude-opus-5-5  ~/src/your-project  1 turn, 1 edit  s_…
… claude-code  claude-opus-5-5  ~/src/your-project  1 turn, 1 edit  s_…
More: lothal-ledger sessions --cursor l.…
```

The edit count is the number of calls that edited, deleted or moved files, one for each call: one Codex `apply_patch` or omp `edit` is one edit. A call that failed, was denied or was never confirmed (Codex sends no confirmation for an `apply_patch` it refused, so the end of the turn closes it as interrupted) changed no file and is not counted. These Claude Code examples use one `Write` per session.

A shell command that wrote lines (`cat >> file`) counts as an edit once a commit holds those lines. A session an agent started for a subtask (a subagent) is listed under the session that started it, indented, with the name it was given and `(subagent of s_…)`.

`search` finds words in the prompts, answers and narration this machine kept (and, signed in, what you may read on the team platform), each hit with its session:

```console
$ lothal-ledger search delays
s_…  …  claude-code  prompt
    add a function «delays»(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of them
```

The text form is the same on a terminal and piped. `--json` prints one JSON document, for a script. For example:

```console
$ lothal-ledger search delays --json | jq -c '{fields: [.hits[].matched] | unique}'
{"fields":["prompt"]}
```

`lothal-ledger stats [<commit or range>]` counts the added lines by state, agent and model.

## How the agent got there: `why --trace` and `trace`

`why <file>:<line> --trace` shows the turn that wrote the line, block by block in time order: your prompt, the model's thinking, its answers, each tool call with its input and output, what each call read and which lines it wrote, with their state and commit:

```console
$ lothal-ledger why backoff.py:5 --trace
Session s_… · claude-code · 1 turn · 1 call · thinking: text
Models: claude-opus-5-5

Turn 1 · … · completed · claude-opus-5-5
  prompt: add a function delays(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of them
  model: - → claude-opus-5-5
  thinking (text): Use the retry budget for the delay sequence.
  call Write […] · ok
    input: {"content":"from retry import RETRY_BUDGET\n\n\ndef delays(base):…
    output: File created successfully at: ~/src/your-project/backoff.py
    wrote: backoff.py 1-5 (ai, …)
```

The header says how much of the model's thinking the agent exposes: `text` when its transcript holds the thinking itself (omp, and Claude Code when it keeps the text), `summaries` (Codex's reasoning summaries), `opaque (encrypted, redacted or signed)` when only a signature, an encrypted block or a provider-redacted one was there, shown by its kind and size and never its bytes, or `not exposed`. A field the ledger does not have the text of says why in its place ([Privacy](privacy.md#what-you-see-instead-of-the-text)).

`trace <session>` shows a whole session that way, turn by turn; `<session>` is the `s_…` reference `sessions`, `why` and the commit trailer print, and `--turn <n>` shows one turn. Here, the newest session:

```console
$ lothal-ledger trace "$(lothal-ledger sessions --limit 1 --json | jq -r '.sessions[0].session_ref')" --turn 1
Session s_… · claude-code · 1 turn · 1 call · thinking: text
Models: claude-opus-5-5

Turn 1 · … · completed · claude-opus-5-5
  prompt: add a function delays(base) in a new file backoff.py: the wait before each retry, doubling from base, RETRY_BUDGET of them
  model: - → claude-opus-5-5
  thinking (text): Use the retry budget for the delay sequence.
…
```

A long field is cut at 600 characters with its full size; `--json` carries up to 16 KiB of it. A long session comes a page at a time (at most 50 turns or 1,000 blocks), each page ending with the command for the next: `More: lothal-ledger trace s_… --cursor …`. Both read this machine's store only.

## For scripts and editors: `--json`

Every command here takes `--json`: one JSON document per answer, with no other text. With a line, `why --json` also carries the card of the function around it (`enclosing_symbol`), and each prompt's `content_state`: `present` when its text follows, otherwise why not, with the sentence for it in `content_message`:

```console
$ lothal-ledger why backoff.py:5 --json | jq -c '{state, content_state: .writers[0].sessions[0].calls[0].content_state, enclosing_symbol: .enclosing_symbol.symbol.name}'
{"state":"ai","content_state":"present","enclosing_symbol":"delays"}
```

`sessions --json` gives a page as `{sessions, next}`; pass `next` back as `--cursor` for the following page:

```console
$ lothal-ledger sessions --limit 2 --cursor "$(lothal-ledger sessions --limit 2 --json | jq -r .next)"
… claude-code  claude-opus-5-5  ~/src/your-project  1 turn, 1 edit  s_…
… claude-code  claude-opus-5-5  ~/src/your-project  1 turn, 1 edit  s_…
```

`--version --json` names the version and the number of the JSON contract the editors check (`2`), and `doctor --json` lists what needs you under `attention`, each with its remedy ([Enable and modes](enable-and-modes.md#check-the-setup)):

```console
$ lothal-ledger --version --json | jq -c '{contract}'
{"contract":2}
$ lothal-ledger doctor --json | jq -c '{status, attention}'
{"status":"ok","attention":[]}
```

## Links to the web: `status --summary --json`

An editor asks `status --summary --json` about the repository it opens: its mode, the sign-in, the upload queue, the state of the store, and `web`, this repository's links into the web app ([On the web](web.md)). `web` holds three URL templates: `why` (a line at a commit), `trace` (a session's whole trace) and `chain` (a function's history); the editor fills in `{commit}`, `{path}`, `{line}`, `{session_hash}` and `{symbol}`. They come from the team platform's answers to this repository's uploads under your account, so `web` is `null` while signed out, in a `local` repository, under another account, and before the repository's first upload. Signed out, as [Sign in and sync](login-and-sync.md#sign-out) left this machine:

```console
$ lothal-ledger status --summary --json | jq -c '{signed_in: .sync.signed_in, web}'
{"signed_in":false,"web":null}
```

Signed in again as the same account, the links are back, from the uploads before:

```console
$ lothal-ledger login
To sign in, open …
and confirm the code …
Waiting for the approval…
Logged in to https://api.dev.lothal.app as you@example.com (org your-org).
…
$ lothal-ledger status --summary --json | jq -r '.web.why, .web.trace, .web.chain'
https://app.dev.lothal.app/r/…/c/{commit}/why?path={path}&line={line}
https://app.dev.lothal.app/r/…/s/{session_hash}/trace
https://app.dev.lothal.app/r/…/c/{commit}/chain?path={path}&symbol={symbol}
```

The rest of this guide is followed signed out:

```console
$ lothal-ledger logout
Logged out of https://api.dev.lothal.app.
…
```

Next: [Inside the agent](in-agent.md).
