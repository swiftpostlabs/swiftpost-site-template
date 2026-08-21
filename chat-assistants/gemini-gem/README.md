# Gemini Gem

A Gem is a saved Gemini configuration with its own instructions and its own
uploaded knowledge files.

## Setting one up

Pick a language folder: [`site-template/`](./site-template/) for English, where
the assistant answers to Mr. Wolf, or [`site-template-it/`](./site-template-it/)
for Italian, where it answers to Ettore. Then:

1. Go to <https://gemini.google.com> and create a new Gem. Look for Gems in the
   sidebar; the entry point has moved around over time.
2. Open `instructions.md` from the folder you picked, copy all of it, and paste
   it into the Gem's instructions field.
3. Upload `js.md` and `dev.md` from the same folder under **Knowledge**.
4. Save, then try it on something small: ask it to add a page at `/about`. A
   good answer names `packages/main/src/app/about/page.tsx`, gives the whole
   file, keeps `'use client'` out of it, and tells you to run `yarn typecheck`.

Each folder also has a generated `README.md` listing exactly what it contains
and which skills were left out of the knowledge files.

## What is in the knowledge files

14 skills, about 14,300 words, in 2 of the 10 files a Gem allows.

| File | Covers |
| --- | --- |
| `js.md` | TypeScript, React, Next.js, this template's Elysium UI library, its Next constraints, and its styling rules |
| `dev.md` | Coding patterns, project structure, this template's monorepo architecture, its packages, and its code conventions |

Left out deliberately: everything about agent tooling (hooks, skills authoring,
retros, task workspaces), CI, Dependabot, versioning, and release management.
Someone editing site files through a chat window cannot act on any of it, and
irrelevant text makes retrieval worse rather than better. There was room for it:
8 of 10 file slots are unused.

## Regenerating

**This repository cannot rebuild them.** Its `agentic-tools` dependency resolves
to the legacy Node CLI, which has `skills` and `policy` but no `export`. The
build runs from an `agentic-tools` Python checkout sitting beside this repo:

```bash
cd ../agentic-tools

uv run python -m agentic_tools.main.cli export build --target gemini-gem \
  --skills-root ../swiftpost-site-template/.agents/skills \
  --config ../swiftpost-site-template/chat-assistants/gemini-gem/templates/site-template.config.json \
  --instructions ../swiftpost-site-template/chat-assistants/gemini-gem/templates/site-template.template.md \
  --out ../swiftpost-site-template/chat-assistants/gemini-gem/site-template

# and again with -it on both the --instructions and --out paths
```

Edit `templates/*.template.md`, never the generated `instructions.md`. The
`{{PERSONA}}` and `{{VERIFICATION}}` placeholders are filled verbatim from
`ref-sp-agents-mr-wolf-persona` and `ref-sp-agents-verification-discipline`, and
editing the output forks that text from the skills that own it.

If the exporter reports that a skill *publishes no canonical block*, this repo's
vendored copy is behind the source. Run `yarn upgrade:agentic-tools` and rebuild.

## Known unknowns

- **The instruction field's length limit is undocumented.** These files are
  11,585 characters in English and 13,017 in Italian. Neither has been rejected,
  but neither has been tested. If one is refused, shorten the prose in the
  template rather than the knowledge-file routing at the bottom, which is the
  part retrieval cannot replace.
- **Whether a Gem can search the web is not documented either.** It does not
  matter much here, since the knowledge files are the intended source and the
  instructions tell the assistant to say when something falls outside them.
- **Knowledge limits** are 10 files and 100 MB each. These use 2 files totalling
  about 100 KB, so there is room to add bundles if the template grows.
