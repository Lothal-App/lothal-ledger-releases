# Privacy: what is recorded, what leaves, who reads it

What the ledger keeps on your machine, what it uploads, and who can read what was uploaded. In short: nothing leaves without your choice; what leaves is redacted on your machine first; and the text you upload can be read only by you and by the people your organisation gives content access, each such read logged.

## What is recorded

In a repository you enabled ([Enable and modes](enable-and-modes.md)), for every session of your agents:

- **The session**: the agent, the model, when it ran, its turns.
- **Each prompt you gave**, and each answer the agent wrote.
- **The model's thinking**, as far as the agent writes it down: its text when the agent's transcript holds it (omp; Claude Code when its transcript keeps the text; Codex's reasoning summaries). When the agent keeps only a signature, encrypts its reasoning (Codex) or the provider redacted it, the ledger records that thinking happened, its kind and its size, never those bytes, and decodes nothing.
- **Each tool call**: what the agent asked the tool (the command, the file, the search) and what the tool answered (its output).
- **The context**: which files, searches, command outputs and instructions the model had been given before each call.
- **Which lines each call wrote**, and the commits those lines went into.
- **The decisions** you and the agent recorded.
- **The agent's own records**: each record of its transcript or database the ledger read, kept once, with what was left out of it (opaque thinking, excluded or too large content) declared beside it. `doctor` checks them against the agent's files ([Troubleshooting](troubleshooting.md#the-round-trip)).

Ignore rules keep content out ([Keep content out](enable-and-modes.md#keep-content-out)): `.lothal-ledger/ignore` in the repository, `.git/lothal/ignore` for every checkout of it, and `ignore` in your user config directory for every repository. An item a rule excludes keeps its hash, its byte count and which rule excluded it, never its text; the rule stays on this machine. A rule that excludes the repository whole (`**`) covers every item, including prompts, thinking and decisions. A line in a repository's file never lifts what your own `ignore` file excludes. An item over the configured item cap (8 MiB by default) is not kept, only its hash and size: `Not kept: too large to keep.`

There is no explicit-decision exception: if the repository, the decision's capture scope or any file it names is excluded, `record_decision` keeps only the summary's and rationale's hashes and byte counts. Attribution and file, symbol and session links remain. The uploader also applies current exclusions to previously stored decision text; excluded prose never uploads.

Nothing is recorded in a repository you did not enable, nor outside a repository. The ledger records no keystrokes and no editor saves, and never says a person wrote a line. On this machine what was recorded is yours: the store under `~/.local/state/lothal-ledger` is readable by your account only, and the CLI, the terminal UI and the editors show you all of it.

An incognito session records nothing from the moment you choose it ([Keep a session out: incognito](enable-and-modes.md#keep-a-session-out-incognito)). `lothal-ledger delete-session` removes a session from this machine entirely ([Delete a session](enable-and-modes.md#delete-a-session)), except a session recorded before an upgrade from 0.4.8 or earlier: the store that upgrade set aside is read only, so there the session is hidden from every answer but stays on disk until you delete `legacy/`. What a `sync` repository already uploaded of a deleted session stays on the team platform for now, and `status` says `team delete: pending (Phase 7)`.

A person's save during a long agent shell call (a script or build) can read `ai`, attributed to that agent. This is an accepted attribution limitation.

## What leaves this machine

| Repository | Signed out | Signed in |
|---|---|---|
| not enabled | nothing (nothing is recorded) | nothing |
| `local` | nothing | nothing |
| `local`, repository excluded whole | nothing | nothing |
| `sync` | nothing: it waits for [`sync --backfill-local`](login-and-sync.md#upload-what-you-recorded-before-signing-in) | what is recorded from then on, redacted first |
| `sync`, repository excluded whole | nothing | attribution, links and byte counts; no excluded text or its unkeyed hashes |

**Redacted first.** Before anything uploads, the prompts, answers, thinking, tool inputs and outputs and context go through one redaction on your machine. It replaces recognised secret formats, random-looking values, email addresses and phone numbers, and patterns your own `redaction.custom` rules name, with markers such as `[REDACTED.secret.1f2e3d4c]`. Values in explicit credential positions—authorization and API-key headers, token/API-key flags and query parameters, credential-named assignments and config keys, and URL credentials—are replaced even when they are hex, UUIDs or short passwords. Empty values and placeholders stay unchanged. An item the rules cannot finish is not uploaded at all rather than uploaded half-redacted: it reads `Not uploaded: the redaction rules failed on it.`

Redaction is not a guarantee that arbitrary text contains no secrets: an unrecognised, encoded or split secret can escape detection. A git commit hash outside a credential position is not treated as a key, and prose merely mentioning a token is not removed.

**Never leaves, in any mode:**

- the values redaction replaced: the original secret is never sent;
- the model's opaque thinking (signatures, encrypted or provider-redacted blocks): only its kind and size;
- anything of a `local` repository, including a question about it: the CLI does not ask the team platform about a `local` repository;
- excluded text, including explicit decision summaries and rationales (see above);
- the exact model requests the local proxy keeps (`lothal-ledger enable --proxy`).

## Who reads what you uploaded

What reaches the team platform is in two tiers:

- **Attribution**: which session, agent and model wrote which lines, the commits, the counts. Every member of your organisation who can see the repository reads it.
- **Content**: prompts, answers, thinking, tool inputs and outputs, context. You read your own uploads. Anyone else reads them only with **content access**: a grant an organisation admin gives a person, which can expire and can be revoked. Owners and admins have no content access of their own; they are granted it like anyone else.

Without content access, the web and the CLI show that the content exists and how big it is, never its text, in words that name the field:

```text
A prompt was recorded. Reading it needs content access from an organisation admin.
Thinking was recorded. Reading it needs content access from an organisation admin.
The tool's output was recorded. Reading it needs content access from an organisation admin.
```

**Every read of someone else's text is logged.** Each time a person reads content another person uploaded (a prompt, a thinking block, a tool output, on the web, in the CLI or in a search), the team platform logs that read, one entry per field, in the same step that hands out the text. Reading your own uploads, and seeing one of the lines above instead of the text, logs nothing. No page shows the log yet.

**Nothing per person.** No view, filter, count or ranking is ever made per person: you see what was written and why, never a person's activity.

API keys (for CI) read attribution only: every question about sessions, decisions, content, the trace or an investigation refuses them.

## What you see instead of the text

Every place that would show a prompt, a thought or a tool output says why it does not, in the words every surface uses:

| You see | Why |
|---|---|
| Not recorded: an ignore rule excludes it. | an ignore rule excludes it: a file it names, or the repository whole (`**`, which `status` shows as `content none`); `status` lists the ignore files in force |
| A prompt was recorded. Reading it needs content access from an organisation admin. | it is in the team store; you have no content access to it |
| Not kept: a capture budget ran out (the store was full, or redaction ran out of time). | redaction ran out of time on it, or an earlier release found your local store full when it captured it (a full store now stops recording instead: [Storage](storage.md)) |
| Not kept: too large to keep. | it was over the item cap (8 MiB by default, [Storage](storage.md)) |
| Not uploaded: the redaction rules failed on it. | withheld whole rather than uploaded half-redacted |
| Kept on the machine that recorded it; not uploaded. | its record was uploaded without its content, which stayed on the uploader's machine (for example a backfill without `--content`) |
| Kept on the team store; removed from this machine. | your store, filling up, removed it after the team platform had it; signed in, it is read from there |
| Removed: past its retention, or deleted. | its retention period ended, or it was deleted |
| Held back: a later scan found something that looks secret; only its uploader can read it. | the team platform's re-scan, with newer rules, found what redaction missed |
| The team store did not answer in time. | ask again |
| No prompt was tied to this call. | the agent wrote without a prompt the ledger could tie to the write |

Next: [Storage](storage.md).
