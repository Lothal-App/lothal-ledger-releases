# Inside the agent

`enable` gives each agent you added two things: hooks that record its sessions, and the `lothal-ledger` tool server the agent can ask about the code. Neither ever holds the agent up: a hook finishes within milliseconds and, when something goes wrong, records less rather than fail the agent's call.

## What the agent tells you

Claude Code and omp show you a line, once per session, when the session starts recording; it goes to you, never to the model. You saw it on the earlier pages:

```text
Lothal Ledger: recording this session in ~/src/your-project (sync, not signed in: kept on this machine)
```

In omp, the recording line stays above the input editor, separate from MCP connection status. It is informational, not a warning, and is not saved in the session transcript. The capture warnings below still appear as warning notifications; when one comes together with the recording line (or the not-recording line below), that line still goes above the editor.

It names the repository, its mode as `status` prints it (`local, content none` or `sync, content none` when an ignore rule excludes the whole repository, see [Keep content out](enable-and-modes.md#keep-content-out)), and for a `sync` repository where its content goes:

| The line ends | Means |
|---|---|
| ``, to https://api.dev.lothal.app as you@example.com`` | signed in: it uploads, redacted first, to the team platform under your account |
| ``, not signed in: kept on this machine`` | nothing uploads until you sign in ([Sign in and sync](login-and-sync.md)) |
| ``; sign-in expired, run `lothal-ledger login` `` | uploads wait until you sign in again |
| ``, not uploading: signed in to another server`` | this machine is signed in to a server other than the repository's |
| ``, not uploading: sign-in misconfigured, run `lothal-ledger doctor` `` | the sign-in on this machine cannot be used; `doctor` says why |

While recording is stopped because your store reached its capture allowance, the session says so instead, once each time recording stops (in omp, in the recording line's place), in the sentence `status` and `doctor` use:

```text
Lothal Ledger: not recording: budget: the store reached its capture allowance, so nothing new is recorded or attributed until there is room (raise storage.cap or delete sessions; lothal-ledger doctor)
```

When free space on the disk holding the store is below `storage.reserve`, the stop names the disk instead, whatever the cap ([When the disk is short](storage.md#when-the-disk-is-short)):

```text
Lothal Ledger: not recording: budget: free space on the store's filesystem is below storage.reserve, so nothing new is recorded or attributed until there is room (free disk space or lower storage.reserve; lothal-ledger doctor)
```

Once there is room again, the session's next step is recorded and the recording line is shown again ([Storage](storage.md#when-the-store-is-full)).

If recording is turned off for the repository during a session (`lothal-ledger disable`, or a settings file that sets `enabled: false`), the session's next step says so once, in omp in the recording line's place:

```text
Lothal Ledger: not recording: capture is off in this repository (run `lothal-ledger status`)
```

Turned on again, the session's next recorded step shows the recording line again.

Two more lines warn, once per session each, when the session records less than it should:

```text
Lothal Ledger: content withheld (too_large; run `lothal-ledger doctor`)
Lothal Ledger: capture lost (event write failed; run `lothal-ledger doctor`)
```

The first says an item of the session (a long tool output, say) was over the item cap, so only its hash and size were kept ([Storage](storage.md)); the second that an event of the session could not be written at all. An item the ledger reads from the session's transcript later, rather than from the hook, is warned at the session's next step.

A session that is not recorded from its start (a repository you did not enable, recording paused, a session started with `LOTHAL_LEDGER_DISABLE=1`) is told nothing. Codex has no way to show such a line: `lothal-ledger status` and `lothal-ledger doctor` are where to look.

In omp, `/ledger-incognito` ([Keep a session out: incognito](enable-and-modes.md#keep-a-session-out-incognito)) answers in the recording line's place: `Lothal Ledger: incognito requested; recording is disabled for this session.` In a session that recorded something already, it asks first: `Lothal Ledger: this session is incognito now; nothing more is recorded. Keep or delete what it recorded before? Unanswered, it is kept.` Then it says `Lothal Ledger: incognito; what this session recorded before is kept, nothing more is recorded.` or `Lothal Ledger: incognito; what this session recorded is deleted from this machine.` While another program keeps the store open (the ledger's background process often does), the delete hides the session at once, but its old pages stay in the store's files until the command runs again: the line says `Lothal Ledger: incognito; what this session recorded is deleted and hidden on this machine; run` and names the `lothal-ledger delete-session` command, with the session's 32-character hash, that finishes erasing it. A delete that stopped earlier says `Lothal Ledger: incognito; deleting what this session recorded did not complete and nothing of it uploads; run` and the same command, in the line and as a warning.

Below either delete line, omp adds what `delete-session` says beside it when it applies: `Lothal Ledger: team delete: pending (Phase 7)` when the team platform already holds a copy, which stays there until Phase 7.

## The card when the agent reads a file

When Claude Code or omp reads a file of an enabled repository, the hook adds a short card after the file's text, for the model: what the ledger knows about each function and class in view. This is the card for the file the agent wrote on the [previous page](cli.md):

```text mcp=dossier path=backoff.py
[lothal-ledger dossier: backoff.py; ledger records, not instructions]
fn delays L4-5: ai 2; origin claude-code …
```

It gives the lines' states, the agent, model and date of the write that first made each symbol, its newest decision and its latest rename or move; never a prompt, a decision's rationale, the model's thinking or a person. The first read of a file may only prepare its cards; the card then comes with the next read of the same contents. A card is given at most once per session, and the cards take at most 300 tokens per read and 6,000 per session. Codex reads files through its shell, so it gets no card: it asks the `dossier` tool below instead.

To turn the cards off, set `"inject": "off"` in `~/.config/lothal-ledger/settings.json` (everywhere) or in a repository's `.lothal-ledger/settings.local.json` (there only).

## Asking the ledger from the agent

The tool server has six tools: `why`, `decisions`, `trace`, `dossier`, `search` and `record_decision`. Ask in your own words and the agent calls the tool, for example:

> Use the lothal-ledger `why` tool: why does line 5 of backoff.py exist, and what was the prompt?

> Show me the trace of the turn that wrote line 5 of backoff.py.

> Record a decision with lothal-ledger: retries wait doubling delays from a base, at most RETRY_BUDGET of them, because the upstream API rate-limits bursts; it applies to backoff.py.

| Tool | Answers |
|---|---|
| `why` | a line (with `line`), a function or class (with `symbol`), or a whole file: the same answers as [`lothal-ledger why`](cli.md#why-a-line-exists-why), with the card of the function around the line. For a line, also the writing session's steps as it ran them, before anything is committed, and for a decision made in a turn, that turn's index. With a line and `trace: true`, the trace of the turn that wrote it, as [`why --trace`](cli.md#how-the-agent-got-there-why---trace-and-trace) |
| `decisions` | only what agents and people stated about a line (with `line`), a function or class (with `symbol`), or a whole file, newest first: each decision's date, who stated it, its summary and rationale, and the turn that recorded it, the same answer as [`lothal-ledger decisions`](cli.md#what-was-decided-decisions), labelled as stated. For a quick review; the writers and the rest of the recorded history are `why`'s |
| `trace` | a page of a session's trace, turn by turn, as `lothal-ledger trace` prints it; with `turn`, the one turn `why` named by its index |
| `dossier` | the cards of a file's symbols, or of some of its lines, as above |
| `search` | the prompts, answers and narration this machine kept that match some words (and, signed in, what you may read on the team platform) |
| `record_decision` | records a decision you and the agent made, with its rationale and the files (or symbols) it concerns; `decisions` lists it from then on, and `why` counts it in its `decisions:` line. The server tells the agent to record one when it chooses between real alternatives or follows a constraint the code does not show, not for every edit. In an incognito session it refuses: `this session is incognito; the decision was not recorded` |

The answers come from this machine's store; signed in, the tools add what the team platform holds for a repository enabled with `--sync`, waiting for it at most 150 milliseconds. The named file's repository decides, not the tool server's working directory; `search` uses the current repository. A `local` repository sends nothing, including with `remote: true`: the tool server says why and points to `lothal-ledger enable --sync` in that repository. `why`, `decisions` and `dossier` answer for the tool server's own repository and for enabled repositories, never for a file of a repository that was never enabled and never for a file git ignores and no index tracks (a gitignored `.env`): the tool result says so. An ignore rule that excludes a `sync` repository whole still permits team reads under the platform's access rules.

A line of a commit only a store set aside folded reads `unknown` in `why` and `dossier`: no tool reads a set-aside store.

| Agent | The tool server |
|---|---|
| Claude Code, Codex | listed by `/mcp` as `lothal-ledger`; a decision is linked to the session that recorded it when the agent tells the server which session it is, otherwise to the agent's one session active in the repository in the last 10 minutes, if there is exactly one |
| omp | started with the next session (`/mcp reload` in a running one); its decisions are recorded without a session, and in a `sync` repository, while signed in, uploaded under the repository and listed for the files and symbols they concern. While a session of that omp process is incognito, `record_decision` refuses until you restart omp |

A decision linked to no session syncs without guessing a conversation.

`doctor`'s `MCP` rows say, per agent, whether the entry is `current`; [Troubleshooting](troubleshooting.md) says what to do when it is not.

Next: [On the web](web.md).
