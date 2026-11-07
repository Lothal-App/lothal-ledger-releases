# Storage

Everything the ledger records is kept on your machine, in `~/.local/state/lothal-ledger/`, whatever the repository's mode. This page is about how much it keeps there and what happens when that runs out.

## The caps and the reserve

- **`storage.cap`**, default 2 GiB: the most the ledger keeps on this machine. Everything in the store counts toward it: the database with the prompts, responses, thinking, tool outputs and context it holds, and the agents' raw records.
- **`storage.reserve`**, default 1 GiB: free disk space the ledger leaves to everything else. When the disk's free space drops below it, the ledger behaves as if the cap were reached.
- **`storage.item_cap`**, default 8 MiB: the logical size limit for one content item, separate from the total store cap. You can set it from 256 KiB to 64 MiB. Over-cap content keeps its hash and byte count with `withheld:too_large`, not its text.

`doctor` shows how full the store is, the cap and where the cap comes from:

```console
$ cd ~/src/your-project
$ lothal-ledger doctor
…
Storage: ok; footprint … of the 2.0 GiB cap (…; cap from default), …
…
```

All three are set in your user settings, `~/.config/lothal-ledger/settings.json`, in bytes (from 256 MiB to 64 GiB for `storage.cap`, 256 KiB to 64 MiB for `storage.item_cap`). A repository's settings cannot change them. Values outside either cap's range are clamped to the nearest bound with a note; a non-integer is ignored with a note and the default applies. To raise the total cap to 4 GiB, keeping the file's other settings:

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

At 80 % of the cap, `doctor` warns. Past the cap, the ledger makes room before it gives anything up:

1. **What the team platform already holds goes first.** In a `sync` repository you are signed in for, content the team platform has acknowledged is removed from this machine, oldest first, from 90 % of the cap down to 70 %. The raw record kept beside it goes the same way, oldest first. Its record stays (which session, which lines, how large, what was left out); `why` then reads the prompt from the team platform, and when that cannot be reached it says the prompt was evicted and the team platform holds it.
2. **Then recording stops.** When nothing acknowledged is left to remove, the ledger records nothing new: no prompt, response, event or new attribution, not even an empty record of one. Nothing that exists only on this machine is ever removed to make room. A commit you make while recording is stopped still links the lines the ledger had already seen an agent write: that link is written into the room kept inside the cap, or, when even that is full, waits and is written once recording resumes.

While recording is stopped, Claude Code and omp say so once in each session (omp shows it in its recording widget):

```text
Lothal Ledger: not recording: budget (the store reached its storage cap; run `lothal-ledger doctor`)
```

`status` and `doctor` say `not recording: budget` and since when; `doctor` names the remedy: raise `storage.cap`, shorten local retention, or `disable` repositories you no longer need recorded. Signing in with `sync` repositories also helps: what the team platform holds can then be removed from this machine.

Once there is room again, recording resumes from that moment. What the agents did during the stop is not read back in later (agent files are not the ledger's content store); `doctor` says when recording stopped and resumed, so the gap is named.

A store an earlier release filled kept event records without their content (`withheld:budget`). `doctor`'s `Storage:` line still counts those until a retention sweep deletes something.

## Retention

Content not uploaded is kept 90 days on this machine and the rest of a record 365 days. Once the team platform has acknowledged an upload, its local copy is kept 7 days and its record 30 days.

Next: [The terminal UI](tui.md).
