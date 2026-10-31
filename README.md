# Development Field Notes

A small, public notebook of practical development lessons: gotchas, fixes, failed approaches, and things worth checking again. It is Markdown-first and deliberately useful as plain files in GitHub; no site build or account is needed to read it.

## Browse and search

- [Notes index](notes/README.md)
- [Note template](templates/note.md)

Anyone can read the repository without signing in. Browse files on GitHub or clone it anonymously and search locally with `rg`. GitHub search may require signing in:

```sh
git clone https://github.com/mitchins/dev-notes.git
cd dev-notes
rg -n "search terms" .
```

Raw files are available without signing in: [agent instructions](https://raw.githubusercontent.com/mitchins/dev-notes/main/AGENTS.md) and [notes index](https://raw.githubusercontent.com/mitchins/dev-notes/main/notes/README.md). The default branch is `main`.

Keep one focused note per learning and link related notes instead of copying them.

## Contribute a note

1. Copy [`templates/note.md`](templates/note.md) into the most relevant topic folder under `notes/`.
2. Give the file a descriptive, searchable name, for example `sdl2-input-focus.md`.
3. State what is confirmed and what is still a hypothesis. Include versions, evidence, and a way to verify the result when available.
4. Add a link to the note in that topic's `README.md` index. Create a topic folder only when there is a real note to put in it.
5. Review the diff for secrets and private details before sharing it.

Notes can describe non-game development too. Porting work, including Analogue Pocket RISC/SDL2, is an initial area of interest; no project-specific findings are assumed here.

## Access and updates

This repository is public: anyone can read it anonymously from GitHub or clone it without credentials. Writing still requires the repository owner's existing GitHub write access and whatever credentials are already configured for that account. Repository instructions do not grant access, bypass permissions, or supply credentials.

For work in another project, first check that project's instructions and use its existing GitHub setup. This copy-pastable pointer can be included in another repository's `AGENTS.md` or agent prompt:

```text
Shared development notes: https://github.com/mitchins/dev-notes
Before solving a tricky issue, search the notes for relevant platform or library gotchas. Read README.md and AGENTS.md before contributing. When explicitly asked to record a learning, capture useful failed approaches as well as verified fixes, with public evidence when available; label hypotheses and never invent claims. Read and clone access are public; contributions require existing GitHub write access. Use this project's existing GitHub setup, preserve unrelated changes, and never force-push.
```

Make changes in a separate clone or branch, fetch/rebase before proposing a contribution, and use a normal commit or pull request. Do not force-push shared branches or overwrite changes you did not make. Separate clones and branches reduce collisions; coordinate if multiple contributors are changing the same note.

## Safety and scope

The contents are public. Do not add credentials, private chat or email content, host details, personal data, or invented claims. Unconfirmed explanations are welcome when clearly labelled as hypotheses. Remove or generalize identifying details. Label uncertain advice clearly, and revisit notes when versions or behavior change.

This is a shared notebook, not authoritative documentation. Keep entries concise, practical, and tied to evidence where possible. The existing [Research site](https://mitchins.github.io/research/) is a separate cross-purpose ML research project and is not part of this repository.
