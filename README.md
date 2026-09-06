# agent-skills

Agent skills for Claude Code and Codex.

## Original work

The skills in `skills/` are original work by Kaniikura, released into the public
domain under [CC0 1.0](LICENSE). Use, adapt, and redistribute them however you
like — no attribution required. A link back is welcome but never expected.

- **`japanese-tech-writing`** — Prose conventions for Japanese technical books and
  articles: formatting, paragraph construction, rigor of argument, reader load,
  narrative stance, and removal of filler.
- **`cognitive-rhythm-writing`** — Designing rhythm and density in expository
  prose, treating pacing as a switch of cognitive mode rather than decoration.
  It reads `japanese-tech-writing` first, so keep both installed.

### Install

```
npx skills add Kaniikura/agent-skills -g -a '*' -s '*' -y
```

## Third-party skills

The skills below are **not** authored by me. They are listed here only to record
my setup and are **not** redistributed in this repository — no file from them is
included. Install them from their original sources.

| Skill | Author | Source | License |
| --- | --- | --- | --- |
| `grilling`, `handoff`, `claude-handoff`, `prototype`, `resolving-merge-conflicts` | Matt Pocock | [mattpocock/skills](https://github.com/mattpocock/skills) | MIT |
| `natural-japanese` | coji | [coji/natural-japanese](https://github.com/coji/natural-japanese) | MIT |
| `visual-explainer` | Nico Bailon | [nicobailon/visual-explainer](https://github.com/nicobailon/visual-explainer) | MIT |
| `explain`, `sanitize-artifacts` | kotek-7 | [kotek-7/dotfiles](https://github.com/kotek-7/dotfiles) | No license (all rights reserved) — listed for reference only, not redistributed |

`.skill-lock.json` records the exact set and versions. Restore the whole
environment with:

```
npx skills experimental_install
```
