# Chat assistants

Setups for consumer chat products, so someone with no coding agent can still get
useful help working on a site built from this template.

These are not skills. Nothing in this folder is loaded by any agent working in
this repository. They are finished text a person pastes into a product's
settings, plus the knowledge files that product uploads.

| Assistant | Product | Language | Folder |
| --- | --- | --- | --- |
| Mr. Wolf | Gemini Gem | English | [`gemini-gem/site-template/`](./gemini-gem/site-template/) |
| Ettore | Gemini Gem | Italian | [`gemini-gem/site-template-it/`](./gemini-gem/site-template-it/) |

One folder is one Gem: the instruction text plus every file to upload. The two
language folders carry identical knowledge files, duplicated on purpose so that
setting one up never means looking in two places.

See [`gemini-gem/README.md`](./gemini-gem/README.md) for setup and for how to
regenerate these.

## What the assistant is for

Someone editing this template by hand, with only a browser and a chat window.
The assistant knows the monorepo layout, where pages and features go, and the
import rules, and it is built around one constraint: **it cannot touch the
repository.** So it gives whole files with full paths, asks to see a file before
rewriting it, and names the command the person should run afterwards instead of
claiming anything works.

## What it is not

It is not a replacement for an agentic tool that can read and edit the repo. It
cannot run `yarn lint`, cannot see whether a file exists, and cannot check that
its own output compiles. Anyone with a real coding agent should point that agent
at `AGENTS.md` and `.agents/skills/` instead, which is a strictly better path.
