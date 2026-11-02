# Storage

Everything the ledger records is kept on your machine, in `~/.local/state/lothal-ledger/`, whatever the repository's mode. This page is about how much it keeps there and what happens when that runs out.

## The cap and the reserve

- **`storage.cap`**, default 2 GiB: the most the ledger keeps on this machine. Everything in the store counts toward it: the database with the prompts, responses, thinking, tool outputs and context it holds, and the agents' raw records.
- **`storage.reserve`**, default 1 GiB: free disk space the ledger leaves to everything else. When the disk's free space drops below it, the ledger behaves as if the cap were reached.

`doctor` shows how full the store is, the cap and where the cap comes from:

```console
$ cd ~/src/your-project
$ lothal-ledger doctor
…
Storage: ok; footprint … of the 2.0 GiB cap (…; cap from default), …
…
```

Both are set in your user settings, `~/.config/lothal-ledger/settings.json`, in bytes (from 256 MiB to 64 GiB for the cap). A repository's settings cannot change them. To raise the cap to 4 GiB, keeping the file's other settings:

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

## When the store is full

At 80 % of the cap, `doctor` warns. Past the cap, the ledger makes room before it gives anything up:

1. **What the team platform already holds goes first.** In a `sync` repository you are signed in for, content the team platform has acknowledged is removed from this machine, oldest first, from 90 % of the cap down to 70 %. Its record stays (which session, which lines); `why` then reads the prompt from the team platform, and when that cannot be reached it says the prompt was evicted and the team platform holds it.
2. **Content still in the agent's own transcript is kept as a pointer.** For agents that write a transcript (Claude Code, Codex, omp), the ledger notes where the text is and stores it once there is room again.
3. **Then new content is withheld.** The record of the session and its lines is still kept, but new prompts and responses are not. Nothing that exists only on this machine is ever removed to make room.

When a session's content is withheld, Claude Code and omp say so once in that session:

```text
Lothal Ledger: content withheld (store budget full; run `lothal-ledger doctor`)
```

and `why` reads `prompt not retained locally` for the lines those sessions wrote. `doctor`'s `Storage:` line counts what was withheld and names the remedy: raise `storage.cap`, shorten local retention, or `disable` repositories you no longer need recorded. Signing in with `sync` repositories also helps: what the team platform holds can then be removed from this machine.

## Retention

Content not uploaded is kept 90 days on this machine and the rest of a record 365 days. Once the team platform has acknowledged an upload, its local copy is kept 7 days and its record 30 days.

Next: [The terminal UI](tui.md).
