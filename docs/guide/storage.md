# Storage

Everything the ledger records is kept on your machine, in `~/.local/state/lothal-ledger/`, whatever the repository's mode. This page is about how much it keeps there and what happens when that runs out.

## The caps and the reserve

- **`storage.cap`**, default 2 GiB: the most the ledger keeps on this machine. One footprint counts toward it: the database with the prompts, responses, thinking, tool outputs, context and agents' own records it holds, its write-ahead log, the content files beside it, and the `.git/lothal` of every repository you enabled. Its logs do not count, and neither does a store an upgrade set aside (see [below](#the-store-set-aside-at-an-upgrade)).
- **The maintenance reserve**: 5 % of the cap, which recording never uses. It keeps room for the ledger's own work (linking commits, eviction, deleting a session) when the store is full. Recording uses the rest, the **capture limit**: 1.9 GiB of the default cap.
- **`storage.reserve`**, default 1 GiB: free disk space the ledger leaves to everything else. When the disk's free space drops below it, recording stops as it does at the capture limit.
- **`storage.item_cap`**, default 8 MiB: the logical size limit for one content item, separate from the total store cap. You can set it from 256 KiB to 64 MiB. Over-cap content keeps its hash and byte count with `withheld:too_large`, not its text, and reads `Not kept: too large to keep.`

`doctor` shows how full the store is, the cap and where the cap comes from, and the maintenance reserve and capture limit:

```console
$ cd ~/src/your-project
$ lothal-ledger doctor
…
Storage: ok; footprint … of the 2.0 GiB cap (…; cap from default), …
…
  maintenance reserve: 102.4 MiB (5 % of cap); capture limit 1.9 GiB; remaining capture allowance …
…
```

`storage.cap`, `storage.reserve` and `storage.item_cap` are set in your user settings, `~/.config/lothal-ledger/settings.json`, in bytes (from 256 MiB to 64 GiB for `storage.cap`, 256 KiB to 64 MiB for `storage.item_cap`); the maintenance reserve is always 5 % of the cap. A repository's settings cannot change them. Values outside either cap's range are clamped to the nearest bound with a note; a non-integer is ignored with a note and the default applies. To raise the total cap to 4 GiB, keeping the file's other settings:

```bash
mkdir -p ~/.config/lothal-ledger && cd ~/.config/lothal-ledger
test -f settings.json || echo '{}' > settings.json
jq '.storage.cap = 4294967296' settings.json > settings.json.new && mv settings.json.new settings.json
cd ~/src/your-project
```

The file then holds, besides anything you had in it:

```json
{
  "storage": {
    "cap": 4294967296
  }
}
```

The next command and the next background pass use it:

```console
$ lothal-ledger status --json | jq -S .storage.cap
{
  "bytes": 4294967296,
  "raised_at_upgrade": null,
  "source": "user"
}
```

`source` is `user` when your settings set the cap, `default` otherwise; `doctor` then says `cap from user`.

To set the logical item cap to 16 MiB, add `"item_cap": 16777216` inside the same `"storage"` object. The cap is cached when the local store is opened: new hook processes use the new value; restart or reopen a long-lived process to apply a change there.

This setting does not raise the server's existing **8 MiB transport object cap**. An item admitted locally is not a promise that a single object of that size can be uploaded; a 64 MiB setting does not enable 64 MiB server objects.

## When the store is full

At 80 % of the cap, `doctor` warns. As the store fills, the ledger makes room before it stops:

1. **What the team platform already holds goes first.** Past 90 % of the cap, content of a `sync` repository that the team platform has acknowledged is removed from this machine, oldest first, down to 70 %. The copy of the agent's own record kept beside it goes the same way once the team platform has taken everything that reads it. Its record stays (which session, which lines, how large, what was left out); `why` then reads the prompt from the team platform, and when that cannot be reached it says the prompt was evicted and the team platform holds it. Nothing that exists only on this machine is ever removed to make room.
2. **Then recording stops, whole.** When the footprint reaches the capture limit, the ledger records nothing new: no prompt, response, event or new attribution, not even an empty record of one. Each step of an agent's work is kept whole or not at all: a step that does not fit is not kept, rather than kept without its text. A commit you make while recording is stopped still links the lines the ledger had already seen an agent write: that link is written into the maintenance reserve, or, when even that is full, waits and is written once recording resumes.

While recording is stopped, Claude Code and omp say so once in each session (omp shows it in its recording widget):

```text
Lothal Ledger: not recording: budget: the store reached its capture allowance, so nothing new is recorded or attributed until there is room (raise storage.cap or delete sessions; lothal-ledger doctor)
```

`status` and `doctor` say it in one sentence, with since when; so do the terminal UI and the VS Code status bar:

```text
not recording: budget: the store reached its capture allowance, so nothing new is recorded or attributed until there is room (raise storage.cap or delete sessions; lothal-ledger doctor)
```

The capture allowance is the cap less the maintenance reserve (`doctor`'s `capture limit`), or nothing once the disk is short of `storage.reserve`.

To make room:

- raise `storage.cap` (above): the capture limit rises with it;
- delete sessions you do not need with `lothal-ledger delete-session` ([Delete a session](enable-and-modes.md#delete-a-session));
- shorten how long content is kept (`retention.local.*`, see [Retention](#retention)): the next background pass deletes what is past it;
- when the disk itself is short, free space on it until more than `storage.reserve` is free.

Recording resumes on its own once 1 % of the cap is free below the capture limit, from that moment. What the agents did during the stop is not read in later; `doctor` names the gap instead: `recording: resumed at …; nothing was recorded from … to then (budget), and that gap is not filled`.

A store an earlier release filled kept event records without their content (`withheld:budget`). `doctor`'s `Storage:` line still counts those until a retention sweep deletes something.

## The store set aside at an upgrade

An upgrade from 0.4.8 or earlier moves the old store, unchanged, into `~/.local/state/lothal-ledger/legacy/` ([Install](install.md#upgrade)). Nothing is recorded into it, nothing is removed from it to make room, and it does not count toward the cap. `status` and `doctor` print where it is, how large it is and how many of its rows were never uploaded:

```text
Legacy store: …/legacy/ledger.db (… bytes, … unsent rows), read-only, set aside at the upgrade; migrated in a later release, in Phase 8
```

`migrated in` says when it moves into the new store (`migrated_in: "phase-8"` in `status --json` and `doctor --json`: a phase, since a release is numbered only when it is cut); its unsent rows wait until then.

## Retention

Content not uploaded is kept 90 days on this machine and the rest of a record 365 days. Once the team platform has acknowledged an upload, its local copy is kept 7 days and its record 30 days.

Next: [The terminal UI](tui.md).
