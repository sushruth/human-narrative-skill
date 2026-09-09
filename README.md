# human-narrative

Writing rules for LLM agents. They cut the tells: uniform sentence length, a hedge on every line, headers and bullets reached for by reflex, "delve," a closing paragraph that restates the one above it.

```bash
npx skills add sushruth/human-narrative-skill
```

Two files. `SKILL.md` is the rules as a skill, invoked per piece of writing, and works in Claude Code, Codex CLI, opencode, Cursor, and Gemini CLI. `output-style.md` is the same rules as a Claude Code output style, applied to a whole session including the model's reasoning.

## Before and after

Same prompt to the Claude CLI (Sonnet) both times: "Write a short paragraph explaining why a team decided to move their background jobs from cron to a task queue." First with nothing else in the system prompt, then with this rule set added. One run each, not cherry-picked from a batch.

**Without:**

> Cron jobs run on a fixed schedule regardless of load, so a burst of work either piles up behind a single run or gets dropped if the job overlaps with itself. A task queue decouples "when work is created" from "when it runs" — jobs get enqueued as they happen, multiple workers pull from the queue in parallel, and failed jobs retry individually instead of rerunning (or silently skipping) an entire batch. That gave the team backpressure, retries, and visibility into per-job failures that cron's fire-and-forget model never had.

**With:**

> Cron jobs kept stepping on each other. A slow job would still be running when the next cron tick fired, so two copies hit the same rows at once and duplicated work. Nothing tracked which jobs had failed, so a bad run just vanished until someone noticed missing data downstream. A task queue fixed both: workers pull one job at a time so overlapping runs stop happening, and failed jobs sit in the queue with a retry count instead of disappearing. It also let the team scale workers independently of the job schedule, instead of tuning cron intervals to guess at load.

The second paragraph invents its details. The prompt asked about a team that doesn't exist, so the overlapping runs and the missing data have no other source. For a PR description the facts are already there and the rules just ask for them to be used. What comes out is still model output. It needs less cleanup.

## What it changes, and what it doesn't

The rules bite on explanatory prose of a paragraph or more: an onboarding doc section, a design note, a "why did we do it this way" answer. There the baseline reaches for sub-headers and bullet lists, hedges with "typically" and "often," and stays at the category level. With the rules it writes paragraphs, names a specific tool or failure, and ends on the real tradeoff instead of an imperative. That held across Claude, Codex, and Gemini CLIs.

Below about three sentences there is nothing for the rules to change. A commit subject, a one-sentence PR description, or a three-line Slack reply comes out the same with or without them. The banned em-dash list ("does X — A, B, C — so that Y") is model-dependent: Codex and Gemini drop it, Sonnet and DeepSeek keep it even with the rule in front of them.

## Install

The `npx skills` command above detects which supported agents are installed and copies `SKILL.md` into each one's skills directory. `-a claude-code -a opencode` targets specific agents; `-g` installs globally instead of per-project.

**By pasting a prompt.** If you'd rather let the agent do it:

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

**By hand.** Every command below uses this:

```bash
RAW=https://raw.githubusercontent.com/sushruth/human-narrative-skill/main
```

Claude Code, Codex CLI, and opencode all read `SKILL.md` from a skills directory. Claude Code reads `~/.claude/skills`, Codex reads `~/.agents/skills`, and opencode reads both, so two copies cover all three:

```bash
mkdir -p ~/.claude/skills/human-narrative ~/.agents/skills/human-narrative
curl -fsSL $RAW/SKILL.md -o ~/.claude/skills/human-narrative/SKILL.md
curl -fsSL $RAW/SKILL.md -o ~/.agents/skills/human-narrative/SKILL.md
```

For one project only, drop the `~` and use `.claude/skills/human-narrative` and `.agents/skills/human-narrative`.

The Claude Code output style is optional and sits on top of the skill:

```bash
mkdir -p ~/.claude/output-styles && curl -fsSL $RAW/output-style.md -o ~/.claude/output-styles/human-narrative.md
```

Then `/output-style human-narrative`.

Claude.ai and Claude Desktop take skills as a zip under Customize → Skills, with code execution enabled:

```bash
curl -fsSL --create-dirs $RAW/SKILL.md -o /tmp/human-narrative/SKILL.md
(cd /tmp && zip -r human-narrative.zip human-narrative)
```

Gemini CLI has no skill system. Append the body of `SKILL.md`, frontmatter stripped, to its context file:

```bash
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> ~/.gemini/GEMINI.md   # global
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> GEMINI.md               # this project only
```

Cursor reads `.cursor/rules/*.mdc`; `.cursorrules` is deprecated:

```bash
mkdir -p .cursor/rules
{ echo "---"; echo "alwaysApply: true"; echo "---"; curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2'; } > .cursor/rules/human-narrative.mdc
```

Anything else that reads `AGENTS.md`:

```bash
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> AGENTS.md
```

## Sources

The rules aren't original. Most restate a finding from one of these papers, two come from writing handbooks, and the banned-word list comes from a blog post. Each entry says which rule it feeds.

- **[StoryScope: Investigating idiosyncrasies in AI fiction](https://arxiv.org/abs/2604.03136)**, Russell, Rajendhran, Pham, Iyyer, Wieting (University of Maryland and Google DeepMind, COLM 2026). Human and AI fiction separate on discourse-level structure (character agency, chronological discontinuity) more than on vocabulary: 93.2% macro-F1 on human-vs-AI detection from narrative features alone, with AI stories clustered tightly and human stories spread out. The skill's name and the "StoryScope-derived rules" line in the output style's frontmatter refer to this paper. It's why the rules treat the tells as structural, and the origin of "break structural symmetry," now part of the paragraph-break rule under Sequence.

- **["Can You Detect the Difference?"](https://arxiv.org/abs/2507.10475)**, Tarım and Onan, 2025. Some citations attach a different title to this arXiv ID, "Feature-Based Detection of AI-Generated Text: An Analysis of Stylometric and Perplexity Markers"; the one above is correct. Measures burstiness, the variance in sentence-level perplexity and length, and finds human writing mixes short and long sentences where autoregressive output stays uniform. Source for "vary sentence length."

- **[A linguistic comparison between human- and AI-generated content](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/)**, Rodrigues, Sturm and Pinheiro, *iScience*, 2026. Same conclusion about rhythm by a different method, LIWC/SAGE stylistic analysis rather than perplexity. Human sentence-length distributions are broad and flat; AI text sits near its own mean.

- **[DetectGPT: Zero-Shot Machine-Generated Text Detection using Probability Curvature](https://arxiv.org/abs/2301.11305)**, Mitchell, Lee, Khazatsky, Manning and Finn, 2023. Model text sits at local maxima of predictability in a model's probability function; human text varies more token to token. The skill can't measure this, but it's the reasoning behind the anti-monotony rules.

- **[Fast-DetectGPT: Efficient Zero-Shot Detection of Machine-Generated Text via Conditional Probability Curvature](https://arxiv.org/abs/2310.05130)**, Bao, Zhao, Teng, Yang and Zhang, 2023. Machine output clusters in predictable word-choice distributions. Predictable word choice tends to be generic word choice, which is the argument behind "be concrete."

- **[Spotting LLMs With Binoculars: Zero-Shot Detection of Machine-Generated Text](https://arxiv.org/abs/2401.12070)**, Hans, Schwarzschild, Cherepanova, Kazemi, Saha, Goldblum, Geiping and Goldstein, 2024. LLM text has low cross-perplexity between two related models; it's easy to predict for a model like the one that wrote it. Same predictability argument, independent method.

- **[Linguistic Characteristics of AI-Generated Text: A Survey](https://arxiv.org/abs/2510.05136)**, Terčon and Dobrovoljc, 2025. Covers structural symmetry, predictable paragraph weighting, reduced lexical diversity, and hedging patterns. Feeds the paragraph-break rule under Sequence, "no flat escalation of hedging," and part of the banned-word list.

- **[Stylometry recognizes human and LLM-generated texts in short samples](https://arxiv.org/abs/2507.00838)**, Przystalski, Argasiński, Grabska-Gradzińska and Ochab, *Expert Systems with Applications*, 2025. Function-word distributions and clause patterns separate human from LLM text in samples as short as ten sentences. The reason the rules apply to a three-line PR comment and not just long documents.

- **["The Science of Scientific Writing"](https://www.americanscientist.org/blog/the-long-view/the-science-of-scientific-writing)**, Gopen and Swan, *American Scientist*, 1990. Readers expect a sentence to open on something already known (the topic position) and close on the new point (the stress position), and they read the grammatical subject as what a passage is about. Source for the Sequence section: chain sentences by topic, and the check that underlines what each sentence is about.

- **Style: Lessons in Clarity and Grace**, Joseph M. Williams (any edition; later ones with Joseph Bizup). The "topic string" chapters: a passage feels coherent when consecutive sentences share a topic, and a topic shift the reader wasn't handed is where they feel a seam. Source for the paragraph-break rule and for re-naming anything that returns from before the last break.

- **["What Are Common Phrases That AI Uses?"](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses)**, DecEptioner (Shadab Sayeed). A blog post, not a paper, and the most literal source here. The banned-word list (delve, testament, tapestry, crucial, pivotal, foster, interplay, landscape, moreover, furthermore, "not only... but also") is taken from its catalog nearly verbatim.
