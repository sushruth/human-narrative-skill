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

- **[StoryScope: Investigating idiosyncrasies in AI fiction](https://arxiv.org/abs/2604.03136)** — Russell, Rajendhran, Pham, Iyyer, Wieting (University of Maryland & Google DeepMind, COLM 2026). AI and human fiction separate on discourse-level narrative structure — character agency, chronological discontinuity — not surface vocabulary: 93.2% macro-F1 on human-vs-AI detection from narrative features alone, with AI stories clustering tightly and human stories spread out. The skill's name and its frontmatter's "StoryScope-derived rules" point here directly; it's why AI-tells get treated as structural rather than lexical, and it grounds the "break structural symmetry" rules.

- **["Can You Detect the Difference?"](https://arxiv.org/abs/2507.10475)** — Tarım & Onan, 2025. (Some citations attach a different title to this arXiv ID — "Feature-Based Detection of AI-Generated Text: An Analysis of Stylometric and Perplexity Markers." The title above is the real one.) Quantifies burstiness — variance in sentence-level perplexity and length — and finds human writing high-burstiness, mixing short and long sentences, while autoregressive AI output stays uniform. Direct source for "alternate short punchy statements with longer multi-clause ones."

- **[A linguistic comparison between human- and AI-generated content](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/)** — Rodrigues, Sturm & Pinheiro, *iScience*, 2026. Same rhythm rule, different method: LIWC/SAGE stylistic analysis instead of perplexity. Human text has a broad, flat sentence-length distribution; AI text clusters near its own mean.

- **[DetectGPT: Zero-Shot Machine-Generated Text Detection using Probability Curvature](https://arxiv.org/abs/2301.11305)** — Mitchell, Lee, Khazatsky, Manning & Finn, 2023. AI text sits at local maxima of predictability in a model's probability function; human text varies far more in token-level predictability. Conceptual footing for the anti-monotony rules — not something the skill can measure directly, but the reason monotony reads as synthetic.

- **[Fast-DetectGPT: Efficient Zero-Shot Detection of Machine-Generated Text via Conditional Probability Curvature](https://arxiv.org/abs/2310.05130)** — Bao, Zhao, Teng, Yang & Zhang, 2023. Machine output clusters in predictable word-choice distributions. Predictable word choice is generic word choice — the argument behind preferring concrete nouns and verbs over generic Latinate abstractions.

- **[Spotting LLMs With Binoculars: Zero-Shot Detection of Machine-Generated Text](https://arxiv.org/abs/2401.12070)** — Hans, Schwarzschild, Cherepanova, Kazemi, Saha, Goldblum, Geiping & Goldstein, 2024. LLM text holds low cross-perplexity between two related models — too easy to predict for a model like the one that wrote it. Same predictability logic, independent method.

- **[Linguistic Characteristics of AI-Generated Text: A Survey](https://arxiv.org/abs/2510.05136)** — Terčon & Dobrovoljc, 2025. Covers structural symmetry, predictable paragraph weighting, reduced lexical diversity, hedging patterns — feeds "break structural symmetry," "no flat escalation of hedging," and part of the vocabulary list at once.

- **[Stylometry recognizes human and LLM-generated texts in short samples](https://arxiv.org/abs/2507.00838)** — Przystalski, Argasiński, Grabska-Gradzińska & Ochab, *Expert Systems with Applications*, 2025. Function-word distributions and clause patterns separate human from LLM text in samples as short as ten sentences; LLM text shows more grammatical standardization. Why the rules apply to a three-line PR comment, not just long documents.

- **["What Are Common Phrases That AI Uses?"](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses)** — DecEptioner (Shadab Sayeed). A blog post, not a paper, and the most literal source here: the anti-cliché list — delve, testament, tapestry, crucial, pivotal, foster, interplay, landscape, moreover, furthermore, "not only... but also" — comes from its catalog almost word for word.
