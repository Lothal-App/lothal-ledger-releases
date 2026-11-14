# Storage

Everything the ledger records is kept on your machine, in `~/.local/state/lothal-ledger/`, whatever the repository's mode. This page is about how much it keeps there and what happens when that runs out.

## The caps and the reserve

- **`storage.cap`**, default 2 GiB: the most the ledger keeps on this machine. One footprint counts toward it: the database with the prompts, responses, thinking, tool outputs, context and agents' own records it holds, the part of its write-ahead log not yet written back into it, the content files beside it, and the `.git/lothal` of every repository you enabled. The same footprint decides when recording stops and when eviction starts, and the ledger gives back the space its files hold beyond it (a written-back log, freed pages), so on disk it takes at most the cap and its maintenance reserve. Its logs do not count, and neither does a store set aside (see [below](#a-store-of-another-build)).
- **The maintenance reserve**: 5 % of the cap, which recording never uses. It keeps room for the ledger's own work (linking commits, eviction, deleting a session) when the store is full. Recording uses the rest, the **capture limit**: 1.9 GiB of the default cap.
- **`storage.reserve`**, default 1 GiB: free disk space the ledger leaves to everything else. When the disk's free space drops below it, recording stops as it does at the capture limit, whatever the cap, and the stop names the disk as its cause ([below](#when-the-disk-is-short)).
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

The capture allowance is the cap less the maintenance reserve (`doctor`'s `capture limit`).

To make room:

- raise `storage.cap` (above): the capture limit rises with it, as long as the disk has that room free beyond `storage.reserve`;
- delete sessions you do not need with `lothal-ledger delete-session` ([Delete a session](enable-and-modes.md#delete-a-session));
- shorten how long content is kept (`retention.local.*`, see [Retention](#retention)): the next background pass deletes what is past it.

`lothal-ledger disable` and `lothal-ledger enable --sync` make no room: they change only what is recorded from then on.

Recording resumes on its own once 1 % of the cap is free below the capture limit, from that moment. What the agents did during the stop is not read in later; `doctor` names the gap instead: `recording: resumed at …; nothing was recorded from … to then (budget), and that gap is not filled`.

### When the disk is short

When free space on the disk holding the store drops below `storage.reserve`, recording stops whatever the cap, and the stop says so instead. Claude Code and omp say:

```text
Lothal Ledger: not recording: budget: free space on the store's filesystem is below storage.reserve, so nothing new is recorded or attributed until there is room (free disk space or lower storage.reserve; lothal-ledger doctor)
```

and `status` and `doctor`:

```text
not recording: budget: free space on the store's filesystem is below storage.reserve, so nothing new is recorded or attributed until there is room (free disk space or lower storage.reserve; lothal-ledger doctor)
```

Raising `storage.cap` does not help then: free space on the disk until more than `storage.reserve` is free, or lower `storage.reserve` in your user settings. Recording resumes on its own once there is. `status --json` names the cause of the stop in force as `storage.stop.cause`: `cap` at the capture limit, `reserve` when the disk is short.

## A store of another build

No build migrates the store you have. When the store on this machine is in another build's schema, after an upgrade or a downgrade, the first command or agent hook moves it, unchanged, into `~/.local/state/lothal-ledger/set-aside/<schema>-<time>/` and starts a fresh store, which records from then on; your settings, your repositories' modes and your agents' hooks are files outside the store and stay as they were. Nothing reads a set-aside store: no answer comes from it, nothing is uploaded from it or removed from it to make room, and it does not count toward the cap. So right after such an upgrade `sessions` can be empty, and what was waiting to upload in the old store is never sent. `status` and `doctor` print one line naming it:

```text
Set-aside store: the store of another build (schema v…; this build uses v…) is in … (…); nothing reads it, and deleting it is safe
```

Deleting the `set-aside/` directory is safe.

## Retention

Content not uploaded is kept 90 days on this machine and the rest of a record 365 days. Once the team platform has acknowledged an upload, its local copy is kept 7 days and its record 30 days.

Next: [The terminal UI](tui.md).
