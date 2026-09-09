# human-narrative

LLM coding agents write explanations that sound like brochures. This skill changes their writing rules for PR descriptions, documentation, code comments, and commit messages.

```bash
npx skills add sushruth/human-narrative-skill
```

The CLI detects installed agents and copies `SKILL.md` into each tool's skill folder. Pass `-g` to install globally, or `-a claude-code -a opencode` to target specific tools.

A Gemini agent running via the `agy` CLI wrote this README in a single run with `human-narrative` loaded, and a person checked the text for invented facts.

## Install

### Skills CLI

```bash
npx skills add sushruth/human-narrative-skill
```

### Manual setup

The source files sit at `https://raw.githubusercontent.com/sushruth/human-narrative-skill/main`.

- **Claude Code**: Save `SKILL.md` to `~/.claude/skills/human-narrative/SKILL.md` (global) or `.claude/skills/human-narrative/SKILL.md` (project).
- **Claude Code output style**: Fetch `output-style.md` to `~/.claude/output-styles/human-narrative.md` and run `/output-style human-narrative`. This applies the rules to the whole session, including the model's reasoning.
- **Codex CLI**: Save `SKILL.md` to `~/.agents/skills/human-narrative/SKILL.md` or `.agents/skills/human-narrative/SKILL.md`.
- **opencode**: Save `SKILL.md` to `~/.config/opencode/skills/human-narrative/SKILL.md`, `.opencode/skills/human-narrative/SKILL.md`, or either of the Claude Code or Codex paths above.
- **Claude.ai / Claude Desktop**: Zip the `human-narrative` folder containing `SKILL.md` and upload it under Customize > Skills with code execution enabled.
- **Cursor**: Save to `.cursor/rules/human-narrative.mdc` with `alwaysApply: true` in the frontmatter and the body of `SKILL.md` below it.
- **Gemini CLI**: Append the body of `SKILL.md` without frontmatter to `~/.gemini/GEMINI.md` or `./GEMINI.md`.
- **Other agents**: Append the body of `SKILL.md` to `AGENTS.md`.

## Sample output

Prompt sent to Claude CLI (Sonnet) in a single run:

> In three or four sentences, explain to a new teammate why the team uses feature flags instead of long-lived feature branches.

| Without rules | With rules |
| :--- | :--- |
| Feature flags let you merge small changes to main continuously, so code stays integrated and tested instead of drifting apart in a branch for weeks. Long-lived branches build up merge conflicts that get worse the longer they live, and they hide bugs until one big scary merge at the end. Flags also let you ship code dark, then turn features on for specific users or roll back instantly without a redeploy — a branch merge has no equivalent "undo." Net effect: smaller diffs, faster feedback, safer rollout. | Long-lived feature branches drift from main for weeks, so merging one back means untangling conflicts against everything else that shipped in the meantime. Feature flags skip that: the code merges to main right away, sits behind a flag, and turns on for real users once it's ready — no big-bang merge, no stale branch. That also means you can ship a half-finished feature dark and test it in production before flipping it on, and roll it back by flipping the flag instead of reverting a merge. The tradeoff is you carry flag-check code and have to remember to delete flags once a feature is fully rolled out — branches don't leave that residue, but they cost more at merge time. |

Without the rules, the model ends on a summary slogan ("Net effect: ..."). With the rules, the model chains each sentence to the previous one and closes on the concrete tradeoff. Both kept the em-dash examples.

## Observed effects

From one run per prompt on each of Claude Code, Codex CLI, opencode, and Gemini CLI, same prompt with and without the rules:

- **Explanatory text (paragraphs or longer)**: Models drop sub-headers, avoid filler hedges ("typically", "often"), write full paragraphs, and finish on real tradeoffs.
- **Short text (under three sentences)**: Commit subjects, single-sentence PR descriptions, and short chat replies show no measurable change.
- **Em-dash list rule**: Codex and Gemini drop mid-sentence em-dash runs. Sonnet and DeepSeek keep them even with the rule present.

## Sources

The rules adapt findings and methods from these sources:

- **Discourse structure**: [StoryScope](https://arxiv.org/abs/2604.03136) (Russell et al., COLM 2026) showed AI and human narrative separate on discourse structure rather than vocabulary.
- **Sentence rhythm**: [Tarım & Onan 2025](https://arxiv.org/abs/2507.10475) and [Rodrigues, Sturm & Pinheiro 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/) measured burstiness and syntactic pacing in human text.
- **Predictability**: [DetectGPT](https://arxiv.org/abs/2301.11305) (Mitchell et al., 2023), [Fast-DetectGPT](https://arxiv.org/abs/2310.05130) (Bao et al., 2023), and [Binoculars](https://arxiv.org/abs/2401.12070) (Hans et al., 2024) tracked local perplexity and token-to-token predictability.
- **Hedging and structural symmetry**: [Terčon & Dobrovoljc 2025](https://arxiv.org/abs/2510.05136) surveyed repetition and hedging markers in model outputs.
- **Short-sample stylometry**: [Przystalski et al. 2025](https://arxiv.org/abs/2507.00838) demonstrated stylometric divergence on ten-sentence samples.
- **Information order**: [Gopen & Swan 1990](https://www.americanscientist.org/blog/the-long-view/the-science-of-scientific-writing) established topic and stress positioning. Williams's *Style: Lessons in Clarity and Grace* provided the rules for topic strings and paragraph seam placement.
- **Banned phrases**: [DecEptioner](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses) provided the inventory of common filler phrases.
