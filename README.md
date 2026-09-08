# human-narrative

A rule set for the writing an agent produces on my behalf: PR descriptions, docs, Slack messages, anything another person reads.

I wrote it because I kept editing the same things out of my own output before sending it. Sentences all about the same length. A hedge on every line. A closing paragraph that restated the one above it. "Delve." None of it was wrong, exactly, but I was rewriting most of it by hand, so I started writing the corrections down as rules instead.

The rules address the mechanical habits. They don't give the text a point of view, or a history with the problem it describes, and a markdown file isn't going to. What's left after applying them is still model output; it just needs less cleanup.

Two files:

- **`SKILL.md`** — the rules as a skill. Invoke it for one piece of writing. Works anywhere that reads `SKILL.md` or an `AGENTS.md`-style context file: Claude Code, Codex CLI, opencode, Cursor, Gemini CLI.
- **`output-style.md`** — the same rules as a Claude Code output style. Claude Code only. Once set it applies to the whole session, reasoning included, not just the final prose.

## Before and after

Same prompt to the Claude CLI both times: "Write a short paragraph explaining why a team decided to move their background jobs from cron to a task queue." First with nothing else in the system prompt, then with this rule set added. One run each, not cherry-picked from a batch. Whether the second one is better is for you to decide.

**Without:**

> A team typically moves background jobs from cron to a task queue once cron's limitations start costing real time: a cron job that fails silently just doesn't run again until the next scheduled tick, and there's no built-in retry, backoff, or alerting to catch that failure automatically. As job volume grows, cron also has no easy way to distribute work across multiple machines or throttle concurrency, so a single overloaded box becomes a bottleneck or single point of failure. A task queue adds visibility — you can see what's pending, running, succeeded, or failed — plus automatic retries with backoff, dead-letter handling for jobs that keep failing, and the ability to scale workers horizontally as load increases. It also enables on-demand and event-triggered jobs (not just fixed-interval ones), which cron can't express well. Teams often make the jump specifically after an incident where a cron failure went unnoticed for hours or days, since that's the moment "it usually works" stops being good enough. The tradeoff is added infrastructure (a broker, worker processes, monitoring) but most teams decide that's worth it for the reliability and observability gains once they're running jobs that matter to the business.

**With:**

> The cron jobs worked fine until they didn't: a single node running the scheduler meant that box going down silently dropped every job scheduled for that window, and nobody found out until a customer asked where their weekly report was. Retries didn't exist either — a job that failed halfway through just failed, with no record of state beyond a stack trace in the logs. As the number of jobs grew, so did the contention: three cron entries all kicking off at 2am would fight over the same database connections, and there was no way to prioritize a time-sensitive job over a bulk cleanup task. Moving to a task queue (they picked Sidekiq) gave them retries with backoff, visibility into what's queued versus running versus failed, and the ability to scale workers horizontally instead of praying one box could keep up. It also let them decouple "when does this get triggered" from "how many workers process it," so a spike in job volume no longer meant everything backed up behind a single cron tick. The migration took about three weeks, mostly rewriting jobs to be idempotent since the queue could now retry them.

One thing worth noticing: the second paragraph invents details (Sidekiq, three weeks, a weekly report). The prompt asked for a story about a team that doesn't exist, so that's the only way to be concrete. For a PR description the facts are already there and the rules just ask for them to be used.

## Install with the skills CLI

If you have [`npx skills`](https://github.com/vercel-labs/skills):

```bash
npx skills add sushruth/human-narrative-skill
```

It detects which supported agents are installed and copies `SKILL.md` into each one's skills directory. `-a claude-code -a opencode` targets specific agents; `-g` installs globally instead of per-project.

## Install by pasting a prompt

If you'd rather let the agent do it, paste this:

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

## Install by hand

```bash
RAW=https://raw.githubusercontent.com/sushruth/human-narrative-skill/main
```

**Claude Code, Codex CLI, opencode.** All three read `SKILL.md` from a skills directory, so it's the same file copied to two places:

```bash
mkdir -p ~/.claude/skills/human-narrative ~/.agents/skills/human-narrative
curl -fsSL $RAW/SKILL.md -o ~/.claude/skills/human-narrative/SKILL.md
curl -fsSL $RAW/SKILL.md -o ~/.agents/skills/human-narrative/SKILL.md
```

- Claude Code: `~/.claude/skills` (global), `.claude/skills` (project).
- Codex CLI: `~/.agents/skills` (global), `.agents/skills` (project).
- opencode: reads both of the above, plus `~/.config/opencode/skills` / `.opencode/skills`. The two commands cover it.

For a project-scoped install, use `.claude/skills/human-narrative` and `.agents/skills/human-narrative` without the `~`.

**Claude Code output style.** Optional, on top of the skill. Applies for the whole session rather than per invocation:

```bash
mkdir -p ~/.claude/output-styles && curl -fsSL $RAW/output-style.md -o ~/.claude/output-styles/human-narrative.md
```

Then `/output-style human-narrative`.

**Claude.ai / Claude Desktop.** Skills upload as a zip under Customize → Skills, with code execution enabled:

```bash
curl -fsSL --create-dirs $RAW/SKILL.md -o /tmp/human-narrative/SKILL.md
(cd /tmp && zip -r human-narrative.zip human-narrative)
```

Upload `/tmp/human-narrative.zip`.

**Gemini CLI.** No skill system; append the body of `SKILL.md` (frontmatter stripped) to its context file:

```bash
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> ~/.gemini/GEMINI.md   # global
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> GEMINI.md               # this project only
```

**Cursor.** Rules live in `.cursor/rules/*.mdc` now; `.cursorrules` is deprecated:

```bash
mkdir -p .cursor/rules
{ echo "---"; echo "alwaysApply: true"; echo "---"; curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2'; } > .cursor/rules/human-narrative.mdc
```

**Anything else that reads `AGENTS.md`:**

```bash
curl -fsSL $RAW/SKILL.md | awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' >> AGENTS.md
```

## Sources

The rules aren't original. Most of them restate a finding from one of these, and a few came from a blog post. Listed with what each one contributed.

- **[StoryScope: Investigating idiosyncrasies in AI fiction](https://arxiv.org/abs/2604.03136)** — Russell, Rajendhran, Pham, Iyyer, Wieting (University of Maryland & Google DeepMind, COLM 2026). Human and AI fiction separate on discourse-level structure (character agency, chronological discontinuity) more than vocabulary: 93.2% macro-F1 on human-vs-AI detection from narrative features alone, with AI stories clustered tightly and human stories spread out. The skill's name and the "StoryScope-derived rules" line in its frontmatter refer to this paper. It's why the rules treat the tells as structural, and where "break structural symmetry" comes from.

- **["Can You Detect the Difference?"](https://arxiv.org/abs/2507.10475)** — Tarım & Onan, 2025. (Some citations attach a different title to this arXiv ID, "Feature-Based Detection of AI-Generated Text: An Analysis of Stylometric and Perplexity Markers." The one above is correct.) Measures burstiness, the variance in sentence-level perplexity and length, and finds human writing mixes short and long sentences where autoregressive output stays uniform. Source for "vary sentence length."

- **[A linguistic comparison between human- and AI-generated content](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/)** — Rodrigues, Sturm & Pinheiro, *iScience*, 2026. Same conclusion about rhythm by a different method, LIWC/SAGE stylistic analysis rather than perplexity. Human sentence-length distributions are broad and flat; AI text sits near its own mean.

- **[DetectGPT: Zero-Shot Machine-Generated Text Detection using Probability Curvature](https://arxiv.org/abs/2301.11305)** — Mitchell, Lee, Khazatsky, Manning & Finn, 2023. Model text sits at local maxima of predictability in a model's probability function; human text varies more token to token. The skill can't measure this, but it's the reasoning behind the anti-monotony rules.

- **[Fast-DetectGPT: Efficient Zero-Shot Detection of Machine-Generated Text via Conditional Probability Curvature](https://arxiv.org/abs/2310.05130)** — Bao, Zhao, Teng, Yang & Zhang, 2023. Machine output clusters in predictable word-choice distributions. Predictable word choice tends to be generic word choice, which is the argument for concrete nouns and verbs over Latinate abstractions.

- **[Spotting LLMs With Binoculars: Zero-Shot Detection of Machine-Generated Text](https://arxiv.org/abs/2401.12070)** — Hans, Schwarzschild, Cherepanova, Kazemi, Saha, Goldblum, Geiping & Goldstein, 2024. LLM text has low cross-perplexity between two related models; it's easy to predict for a model like the one that wrote it. Same predictability argument, independent method.

- **[Linguistic Characteristics of AI-Generated Text: A Survey](https://arxiv.org/abs/2510.05136)** — Terčon & Dobrovoljc, 2025. Covers structural symmetry, predictable paragraph weighting, reduced lexical diversity, and hedging patterns. Feeds "break structural symmetry," "no flat escalation of hedging," and part of the banned-word list.

- **[Stylometry recognizes human and LLM-generated texts in short samples](https://arxiv.org/abs/2507.00838)** — Przystalski, Argasiński, Grabska-Gradzińska & Ochab, *Expert Systems with Applications*, 2025. Function-word distributions and clause patterns separate human from LLM text in samples as short as ten sentences. The reason the rules apply to a three-line PR comment and not just long documents.

- **["What Are Common Phrases That AI Uses?"](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses)** — DecEptioner (Shadab Sayeed). A blog post, not a paper, and the most literal source here. The banned-word list (delve, testament, tapestry, crucial, pivotal, foster, interplay, landscape, moreover, furthermore, "not only... but also") is taken from its catalog nearly verbatim.
