# human-narrative

A prose/reasoning style rule set that strips AI writing tells — moralizing endings, hedge-ratcheting, uniform sentence rhythm, the "delve/crucial/tapestry" vocabulary cluster — and replaces them with plain, direct, human-register output. Ships in two forms:

- **`SKILL.md`** — a Claude Code skill, invoked on demand for a specific writing task (PR description, doc, message).
- **`output-style.md`** — a Claude Code output style, applied for the whole session including internal reasoning, not just the final text.

Same rule set in both. The split exists because "govern this one piece of prose" and "govern how I think and talk for the rest of this session" are different scopes — the always-on style covers the one the on-demand skill can't.

## Install (automated)

Paste this to your agent and let it do the rest:

```
Install the "human-narrative" skill from https://github.com/sushruth/human-narrative-skill.
Fetch https://raw.githubusercontent.com/sushruth/human-narrative-skill/main/SKILL.md and save
it as SKILL.md inside a human-narrative/ folder under whichever of these you have available:
~/.claude/skills/, ~/.agents/skills/, ~/.config/opencode/skills/ (global), or .claude/skills/,
.agents/skills/, .opencode/skills/ (this project only). Create directories as needed. If this
is Claude Code, also fetch https://raw.githubusercontent.com/sushruth/human-narrative-skill/main/output-style.md
into ~/.claude/output-styles/human-narrative.md and tell me to run /output-style human-narrative.
If this is Cursor, save SKILL.md's body (without its YAML frontmatter) into
.cursor/rules/human-narrative.mdc with a frontmatter block of just `alwaysApply: true`. If
none of those apply, append the body (without frontmatter) to AGENTS.md or GEMINI.md instead.
```

## Install (manual)

```bash
RAW=https://raw.githubusercontent.com/sushruth/human-narrative-skill/main
```

**Claude Code, Codex CLI, and opencode all discover `SKILL.md` natively** — install once per tool, same file:
```bash
mkdir -p ~/.claude/skills/human-narrative ~/.agents/skills/human-narrative
curl -fsSL $RAW/SKILL.md -o ~/.claude/skills/human-narrative/SKILL.md
curl -fsSL $RAW/SKILL.md -o ~/.agents/skills/human-narrative/SKILL.md
```
- Claude Code reads `~/.claude/skills` (global) / `.claude/skills` (this project).
- Codex CLI reads `~/.agents/skills` (global) / `.agents/skills` (this project).
- opencode reads *both* of those plus its own `~/.config/opencode/skills` / `.opencode/skills` — the two commands above already cover it, nothing extra needed.

Project-scoped instead of global: same two commands with `.claude/skills/human-narrative` and `.agents/skills/human-narrative` (drop the `~`).

**Claude Code — output style** (session-wide, governs reasoning too, not just prose):
```bash
mkdir -p ~/.claude/output-styles && curl -fsSL $RAW/output-style.md -o ~/.claude/output-styles/human-narrative.md
```
Then `/output-style human-narrative`.

**Claude.ai / Claude Desktop** — skills there are uploaded as a zip (needs Code execution enabled, under Customize → Skills):
```bash
curl -fsSL --create-dirs $RAW/SKILL.md -o /tmp/human-narrative/SKILL.md
(cd /tmp && zip -r human-narrative.zip human-narrative)
```
Upload the resulting `/tmp/human-narrative.zip`.

**Gemini CLI** (no native skill system — append to its context file instead):
```bash
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> ~/.gemini/GEMINI.md   # global
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> GEMINI.md               # this project only
```

**Cursor** (current format is `.cursor/rules/*.mdc`; the old `.cursorrules` file is deprecated):
```bash
mkdir -p .cursor/rules
{ echo "---"; echo "alwaysApply: true"; echo "---"; curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2'; } > .cursor/rules/human-narrative.mdc
```

**Any other AGENTS.md-style agent:**
```bash
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> AGENTS.md
```

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
