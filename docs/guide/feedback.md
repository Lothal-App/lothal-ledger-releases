# Feedback

Report problems and ideas as issues on the public repository the programs come from: [`Lothal-App/lothal-ledger-releases`](https://github.com/Lothal-App/lothal-ledger-releases/issues/new). One problem per issue.

## What to include

- What you did, what you expected, and what happened instead, with the commands you ran and what they printed.
- `lothal-ledger --version`, your operating system, and the agent and its version (`claude --version`, `codex --version`, `omp --version`).
- What `lothal-ledger doctor` prints, run in the repository where it happened.
- For a wrong answer (a line attributed to a session that did not write it, a line read `ai` that an agent did not write): the file and line, and what `lothal-ledger why <file>:<line>` printed.

## What never to include

The output of `doctor`, `status` and `why` names paths on your machine, your repositories and, for `why`, your prompts. Read it before you paste it, and replace what you do not want public: your home directory, repository names, prompt text. Never paste a sign-in code, an API key (`llk_…`) or the contents of `~/.local/state/lothal-ledger/auth/`. A wrong attribution can be reported without the prompt: the session reference (`s_…`) and the line's state are enough to start.

## What is most useful now

- Anything that blocked you from following this guide on a fresh machine.
- A line attributed wrongly: an agent credited with a line a person wrote, or the wrong session.
- Anything that left your machine that the [modes](enable-and-modes.md#the-modes) say should not have.
- Messages you did not understand.

Back to [the guide](README.md).
