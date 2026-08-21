# Gemini Gem export

Generated. Re-run the exporter rather than editing these files by hand.

## How to use

1. Create a Gem at <https://gemini.google.com>.
2. Paste `instructions.md` into the Gem's instructions field. Its length limit is undocumented; if it is rejected, shorten the prose rather than the routing block — routing is the part retrieval cannot replace.
3. Upload the 2 `.md` files below under **Knowledge**.

## Contents

| File | Skills | Words | KB |
| --- | --- | --- | --- |
| `js.md` | 8 | 9,328 | 66 |
| `dev.md` | 6 | 4,955 | 36 |

**14 skills, 14,283 words** across 2 of 10 allowed knowledge files.

## Excluded, and why

| Skill | Reason |
| --- | --- |
| `ref-sp-agents-adversarial-review` | domain agents is not in the requested domains |
| `ref-sp-agents-hooks` | domain agents is not in the requested domains |
| `ref-sp-agents-instructions-authoring` | domain agents is not in the requested domains |
| `ref-sp-agents-local-tasks` | domain agents is not in the requested domains |
| `ref-sp-agents-mr-wolf-persona` | domain agents is not in the requested domains |
| `ref-sp-agents-retro` | domain agents is not in the requested domains |
| `ref-sp-agents-security` | domain agents is not in the requested domains |
| `ref-sp-agents-shareable-skills` | domain agents is not in the requested domains |
| `ref-sp-agents-skills-authoring` | domain agents is not in the requested domains |
| `ref-sp-agents-verification-discipline` | domain agents is not in the requested domains |
| `ref-sp-dev-docs-authoring` | README authoring is not what this assistant is for |
| `ref-sp-dev-git-commits` | the person is not driving git from this chat |
| `ref-sp-dev-github-actions-ci` | CI workflows are out of scope for editing site files |
| `ref-sp-dev-github-dependabot` | dependency automation is out of scope |
| `ref-sp-dev-package-management` | release metadata is out of scope |
| `ref-sp-dev-semantic-versioning` | release numbering is out of scope |
| `tool-sp-commit` | prefix tool- is not exported |
| `tool-sp-create-skill` | prefix tool- is not exported |
| `tool-sp-handle-agents-local-tasks` | prefix tool- is not exported |
| `tool-sp-maintain-agents-instructions` | prefix tool- is not exported |
| `tool-sp-maintain-skills` | prefix tool- is not exported |
| `tool-sp-make-skill-shareable` | prefix tool- is not exported |
| `tool-sp-run-adversarial-review` | prefix tool- is not exported |
| `tool-sp-setup-agent-repo` | prefix tool- is not exported |
| `tool-spst-adopt-template` | prefix tool- is not exported |

## Limits this export respects

- 10 knowledge files per Gem, 100 MB per file (<https://support.google.com/gemini/answer/14903178>).
- Bundles are grouped by skill domain for retrieval coherence, not packed by size.
- Output directory: `../swiftpost-site-template/chat-assistants/gemini-gem/site-template-it`.
