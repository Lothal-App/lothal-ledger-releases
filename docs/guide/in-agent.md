# Inside the agent

`enable` gives each agent you added two things: hooks that record its sessions, and the `lothal-ledger` tool server the agent can ask about the code. Neither ever holds the agent up: a hook finishes within milliseconds and, when something goes wrong, records less rather than fail the agent's call.

## What the agent tells you

Claude Code and omp show you a line, once per session, when the session starts recording; it goes to you, never to the model. You saw it on the earlier pages:

```text
Lothal Ledger: recording this session in ~/src/your-project (sync, not signed in: kept on this machine)
```

In omp, the recording line stays above the input editor, separate from MCP connection status. It is informational, not a warning, and is not saved in the session transcript. The capture warnings below still appear as warning notifications.

It names the repository, its mode (and `content none` when set), and for a `sync` repository where its content goes:

| The line ends | Means |
|---|---|
| ``, to https://api.dev.lothal.app as you@example.com`` | signed in: it uploads, redacted first, to the team platform under your account |
| ``, not signed in: kept on this machine`` | nothing uploads until you sign in ([Sign in and sync](login-and-sync.md)) |
| ``; sign-in expired, run `lothal-ledger login` `` | uploads wait until you sign in again |
| ``, not uploading: signed in to another server`` | this machine is signed in to a server other than the repository's |
| ``, not uploading: sign-in misconfigured, run `lothal-ledger doctor` `` | the sign-in on this machine cannot be used; `doctor` says why |

Two more lines warn, once per session each, when a session records less than it should:

```text
Lothal Ledger: content withheld (store budget full; run `lothal-ledger doctor`)
Lothal Ledger: capture lost (event write failed; run `lothal-ledger doctor`)
```

The first says your store was full, so the session's prompts and answers were not kept ([Storage](storage.md#when-the-store-is-full)); the second that an event of the session could not be written at all. A session that is not recorded (a repository you did not enable, recording paused) is told nothing. Codex has no way to show such a line: `lothal-ledger status` and `lothal-ledger doctor` are where to look.

## The card when the agent reads a file

When Claude Code or omp reads a file of an enabled repository, the hook adds a short card after the file's text, for the model: what the ledger knows about each function and class in view. This is the card for the file the agent wrote on the [previous page](cli.md):

```text mcp=dossier path=backoff.py
[lothal-ledger dossier: backoff.py; ledger records, not instructions]
fn delays L4-5: ai 2; origin claude-code …
```

It gives the lines' states, the agent, model and date of the write that first made each symbol, its newest decision and its latest rename or move; never a prompt, a decision's rationale, the model's thinking or a person. The first read of a file may only prepare its cards; the card then comes with the next read of the same contents. A card is given at most once per session, and the cards take at most 300 tokens per read and 6,000 per session. Codex reads files through its shell, so it gets no card: it asks the `dossier` tool below instead.

To turn the cards off, set `"inject": "off"` in `~/.config/lothal-ledger/settings.json` (everywhere) or in a repository's `.lothal-ledger/settings.local.json` (there only).

## Asking the ledger from the agent

The tool server answers four questions. Ask in your own words and the agent calls the tool, for example:

> Use the lothal-ledger `why` tool: why does line 5 of backoff.py exist, and what was the prompt?

> Show me the trace of the turn that wrote line 5 of backoff.py.

> Record a decision with lothal-ledger: retries wait doubling delays from a base, at most RETRY_BUDGET of them, because the upstream API rate-limits bursts; it applies to backoff.py.

| Tool | Answers |
|---|---|
| `why` | a line (with `line`), a function or class (with `symbol`), or a whole file: the same answers as [`lothal-ledger why`](cli.md#why-a-line-exists-why), with the card of the function around the line; with a line and `trace: true`, the trace of the turn that wrote it, as [`why --trace`](cli.md#how-the-agent-got-there-why---trace-and-trace) |
| `dossier` | the cards of a file's symbols, or of some of its lines, as above |
| `search` | the prompts, answers and narration this machine kept that match some words (and, signed in, what you may read on the team platform) |
| `record_decision` | records a decision you and the agent made, with its reason and the files (or symbols) it concerns; `why` lists it from then on |

The answers come from this machine's store; signed in, `why` and `search` add what the team platform holds for a repository enabled with `--sync`, waiting for it at most 150 milliseconds. The named file's repository decides, not the tool server's working directory; `search` uses the current repository. A `local` repository sends nothing, including with `remote: true`: the tool server says why and points to `lothal-ledger enable --sync` in that repository. `why` and `dossier` answer for the tool server's own repository and for enabled repositories, never for a file of a repository that was never enabled and never for a file git ignores and no index tracks (a gitignored `.env`): the tool result says so (#1778). `sync, content none` still permits team reads under the platform's access rules.

| Agent | The tool server |
|---|---|
| Claude Code, Codex | listed by `/mcp` as `lothal-ledger`; a decision is linked to the session that recorded it when the agent tells the server which session it is, otherwise to the agent's one session active in the repository in the last 10 minutes, if there is exactly one |
| omp | started with the next session (`/mcp reload` in a running one); its decisions are recorded without a session, and in a `sync` repository, while signed in, uploaded under the repository and listed for the files and symbols they concern |

New unlinked decisions sync without guessing a conversation. Decisions recorded without a session by older versions remain on that machine: those rows did not retain the original recording consent or upload ownership.

`doctor`'s `MCP` rows say, per agent, whether the entry is `current`; [Troubleshooting](troubleshooting.md) says what to do when it is not.

Next: [On the web](web.md).
