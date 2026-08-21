# human-narrative

A prose/reasoning style rule set that strips AI writing tells — moralizing endings, hedge-ratcheting, uniform sentence rhythm, the "delve/crucial/tapestry" vocabulary cluster — and replaces them with plain, direct, human-register output. Ships in two forms:

- **`SKILL.md`** — a Claude Code skill, invoked on demand for a specific writing task (PR description, doc, message).
- **`output-style.md`** — a Claude Code output style, applied for the whole session including internal reasoning, not just the final text.

Same rule set in both. The split exists because "govern this one piece of prose" and "govern how I think and talk for the rest of this session" are different scopes — the always-on style covers the one the on-demand skill can't.

## Install

Plain Markdown with YAML frontmatter — portable to anything that reads a file as system-level instructions.

### Claude Code

**As a skill** (on demand, or auto-triggered when the description matches):
```bash
mkdir -p ~/.claude/skills/human-narrative
cp SKILL.md ~/.claude/skills/human-narrative/SKILL.md
```
Project-scoped instead of user-scoped: same file at `.claude/skills/human-narrative/SKILL.md` inside the repo.

**As an output style** (session-wide, reasoning included):
```bash
mkdir -p ~/.claude/output-styles
cp output-style.md ~/.claude/output-styles/human-narrative.md
```
Then `/output-style` → pick `Human Narrative`, or `/output-style human-narrative` directly.

### Claude (claude.ai / Claude Desktop)

Settings → Capabilities → Skills → upload `SKILL.md`. No output-style equivalent there; the on-demand skill is the closest fit.

### Codex CLI / other AGENTS.md-driven agents

Paste the body of `SKILL.md` (below the frontmatter) into `AGENTS.md`. No skill-loading mechanism, so it runs always-on instead of on-demand.

### Gemini CLI

Same move, into `GEMINI.md`.

### Cursor / any rules-file agent

Save as a project rule, e.g. `.cursor/rules/human-narrative.mdc`.

Anything else: drop the rules (minus frontmatter) into whatever file that tool treats as a system prompt or persistent context.

## Sources

Two things formed this.

**[StoryScope: Investigating idiosyncrasies in AI fiction](https://arxiv.org/abs/2604.03136)** — Jenna Russell, Rishanth Rajendhran, Chau Minh Pham, Mohit Iyyer, John Wieting (University of Maryland & Google DeepMind, COLM 2026). Shows AI and human fiction separate cleanly on discourse-level narrative choices — character agency, chronological structure — rather than surface style: 93.2% macro-F1 on human-vs-AI detection from narrative features alone, with AI stories clustering tightly while human stories spread out. This is the frontmatter's "StoryScope-derived rules" and the reason the skill targets structural sameness, not just banned words.

**A second skill, `human-thought-and-expression`**, merged in later, brought in the AI-text-detection/stylometry literature below. Checked each against the actual paper rather than trusting the list at face value:

| Citation | Status |
|---|---|
| [DetectGPT](https://arxiv.org/abs/2301.11305) — Mitchell et al., 2023 | Confirmed |
| [Fast-DetectGPT](https://arxiv.org/abs/2310.05130) — Bao et al., 2023 | Confirmed |
| [Binoculars](https://arxiv.org/abs/2401.12070) — Hans et al., 2024 | Confirmed |
| ["Feature-Based Detection of AI-Generated Text..."](https://arxiv.org/abs/2507.10475) — Tarım & Onan, 2025 | **Title fabricated.** Real title: "Can You Detect the Difference?" — same authors/ID/year, narrower paper (diffusion vs. autoregressive text, not a general stylometry survey) |
| [A Linguistic Comparison Between Human- and AI-Generated Content](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/) — Rodrigues, Sturm & Pinheiro, *iScience*, 2026 | Confirmed |
| [Linguistic Characteristics of AI-Generated Text: A Survey](https://arxiv.org/abs/2510.05136) — Terčon & Dobrovoljc, 2025 | Confirmed |
| [Stylometry Recognizes Human and LLM-Generated Texts in Short Samples](https://arxiv.org/abs/2507.00838) — Przystalski, Argasiński, Grabska-Gradzińska & Ochab, 2025 | Confirmed |
| [What Are Common Phrases That AI Uses?](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses) — DecEptioner | Real, but a blog post, not a paper |

Seven of eight check out. One has the right authors, arXiv ID, and year attached to a title that doesn't exist.
