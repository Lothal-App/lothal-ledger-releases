# Sign in and sync

Signing in connects this machine to the team platform, so that what your `sync` repositories record reaches your organisation's copy and your teammates' questions can be answered from it. Signing in uploads nothing by itself: only a repository you enabled with `--sync` uploads, only what it records while you are signed in, and only after redacting it on this machine.

## A repository that syncs

Enable the repository with `--sync` (add the agents you use; the ones added before are kept):

```console
$ cd ~/src/your-project
$ lothal-ledger enable --sync --agent claude-code
{"slug":"claude-code",…,"status":"installed",…}
…
{"slug":"repository",…,"capture":"sync",…,"status":"enabled",…,"note":"captured, and uploaded to the team while logged in (`lothal-ledger login`); a login never uploads what was captured before",…}
```

Work with your agent as usual. Claude Code and omp say once, at the start of a session, that it is being recorded and where it goes. Not signed in yet, a `sync` repository keeps everything on this machine:

```console
$ claude
Lothal Ledger: recording this session in ~/src/your-project (sync, not signed in: kept on this machine)
```

## Sign in

```console
$ lothal-ledger login
To sign in, open …
and confirm the code …
Waiting for the approval…
Logged in to https://api.dev.lothal.app as you@example.com (org your-org).
…
```

`login` prints a link and a code: open the link in a browser (on a desktop it opens one for you), sign in, and confirm the code. It waits until you do. It signs in to the dev stage, the alpha's team platform, with the dev stage's sign-in application; nothing else to type. If you belong to several organisations, `login` asks which one (or pass `--org <name>`).

The account stays signed in: the ledger refreshes the sign-in by itself. When that is no longer possible, Claude Code's and omp's session line ends with `sign-in expired, run lothal-ledger login`, and `status` says why uploads are paused; run the same `login` command again. Nothing recorded meanwhile is lost: it waits and uploads after the next sign-in.

`status` shows where uploads go and what is waiting:

```console
$ lothal-ledger status
…
Repositories: 1 enabled
  ~/src/your-project: sync
This repository (~/src/your-project): sync
Sync: signed in as you@example.com to https://api.dev.lothal.app (dev stage)
Queue: … pending, … held, … dead, … withheld
Last upload: …
Flusher: …
```

## What uploads, and when

From now on, a session in a `sync` repository uploads as it goes:

```console
$ claude
Lothal Ledger: recording this session in ~/src/your-project (sync, to https://api.dev.lothal.app as you@example.com)
```

- **Only `sync` repositories, only while signed in.** A `local` repository uploads nothing, ever, signed in or not. A repository that is not enabled records nothing at all.
- **Redacted on your machine first.** Secrets (keys, tokens, passwords) and personal data the redaction rules find are replaced before anything leaves; an item the rules cannot process is not uploaded at all.
- **In the background.** Your agent's hooks only write to the local store; a background process (`lothal-ledger flusher`) uploads in batches, a couple of seconds after the first event of a batch or at the end of each turn, and exits on its own when things are quiet. When the team platform cannot be reached, what is waiting stays queued on this machine and the upload is tried again later.
- **Repositories excluded whole** upload attribution, links and byte counts only, never excluded text or its unkeyed hashes. Explicit decision summaries and rationales have no exception (see [Privacy](privacy.md)); current exclusions also apply to already-queued text.
- **What you record while signed out never uploads by itself**, not even after you sign in. That is the next section.

Signed in, `lothal-ledger why`, `trace`, `sessions` and `search` complete what this machine cannot answer from the team platform, for lines and sessions recorded on your teammates' machines. The repository being read must be enabled with `--sync`: for an absolute file path or a recorded session, its repository decides, not the directory you run the command from. A `local` or not-enabled repository sends nothing, even with `--remote` or `--all-users`; the command says why and points to `lothal-ledger enable --sync` in that repository. A `sync` repository excluded whole (`sync, content none` in `status`) still permits team reads; prompts there follow the team platform's rules: you read your own, and a teammate's only with content access your organisation grants.

## Upload what you recorded before signing in

The session you ran before signing in stayed on this machine. `sync --backfill-local` uploads such sessions of your `sync` repositories, after showing you what would go. The preview makes no connection; nothing is queued until you confirm:

```console
$ lothal-ledger sync --backfill-local
sync --backfill-local: preview (envelopes only, content withheld; --content sends it too; no connection was made, nothing is queued before you confirm)

~/src/your-project (sync)
  will go: … events, … bytes

total: … events, … bytes, to https://api.dev.lothal.app
Queue these … events for upload? [y/N] y
queued … events; the flusher uploads them (`lothal-ledger doctor` shows the queue)
```

Without `--content`, only the envelopes go (which session, which turn, which lines; the content withheld); `--content` sends the prompts, responses, the model's thinking, tool outputs and context too, redacted exactly as live uploads are. The preview lists, per repository, what will go and what cannot (content never stored, content swept by local retention, content of a repository excluded whole (`content none`), content withheld by redaction, sessions recorded in `local` mode). `--since 7d` limits it to recent sessions, `--repo <path>` to one repository, and `--yes` skips the question (for scripts).

### Sessions from before you installed

`lothal-ledger import` brings your agents' older sessions into the local store, and keeps them there: an import never uploads, even signed in. To send them to the team platform as well, add `--imported`: `lothal-ledger sync --backfill-local --imported --content` previews, per `sync` repository, the imported sessions (and any recorded before the repository's mode was saved) with their sizes, and queues them only when you confirm. Only sessions whose working directory is in that repository go, only ones this machine recorded, and never one from a time the repository was `local`; a repository excluded whole sends their attribution only. With `--content` it also sends content the team platform acknowledged a session without, when this machine still has it, once. A run queues at most 50,000 events; run it again to send the rest. It sends what the local store holds and never reads your agent's session files: what was not recorded while the store was full ([Storage](storage.md)) is not sent, and the store an upgrade set aside ([Install](install.md#upgrade)) is not read.

## Sign out

```console
$ lothal-ledger logout
Logged out of https://api.dev.lothal.app.
…
```

Signing out stops uploads at once. What was waiting to upload stays on this machine and uploads after you sign in again as the same account in the same organisation. Repositories keep their modes; from now on they record on this machine only.

For CI or a shared machine, an organisation admin can give you an API key (`llk_…`): `lothal-ledger login --with-token` reads it from standard input instead of the browser sign-in.

Next: [Privacy](privacy.md).
