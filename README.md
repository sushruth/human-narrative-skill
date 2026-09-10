# human-narrative

Text written by an agent costs more to read than it needs to. This skill changes how the agent writes anything a person will read, so the information arrives in the order the reader needs it and each sentence follows from the last. The result is noticeably easier to follow.

```bash
npx skills add sushruth/human-narrative-skill
```

The command installs the skill into every coding agent it finds on the machine. Add `-g` to install under your home folder instead of the current project, or `-a claude-code` to pick one agent. To remove it, delete the `human-narrative` folder it created.

## What changes

Replies, docs, PR descriptions, code comments, and commit messages come out in a different shape.

- The first line says where the reader is. What they came for is next. Explanation comes after, if at all.
- Facts are stated flat. Judgments carry a size word. "Verified" appears only when something was run.
- Nothing ends on a moral or a summary, and open questions are left open.
- The usual filler words are banned, along with the mid-sentence run of examples between em-dashes.

Code is left alone. The rules apply to prose, and the Claude Code output style keeps the coding instructions in place.

The skill is not built to pass AI detectors. Several of its sources are detector papers, and they are used to describe what model text does, not as a target to beat.

The skill file is about 16 KB. The agent carries all of it in context while the skill is active.

## Side by side

Same model, same prompt, one run with the skill loaded and one without. Neither output is edited.

**Prompt:** Write the opening of the README for `envcheck`, a CLI that compares a project's .env file against its .env.example and reports keys that are missing or extra. Two or three sentences, then the install command.

Without the skill:

> `envcheck` is a small command-line tool that compares your project's `.env` file against `.env.example` and reports any keys that are missing or no longer needed. Run it before you deploy or onboard a teammate to catch configuration drift before it causes a runtime surprise.
>
> ```
> pipx install envcheck
> ```

With the skill:

> Config files drift from their examples without anyone noticing until something breaks. envcheck reads both files and lists what's missing or extra.
>
> ```
> pipx install envcheck
> ```

The first opens with what the tool is. The second opens with the situation the reader is already in, then says what the tool does about it.

## Install by hand

Both files are at `https://raw.githubusercontent.com/sushruth/human-narrative-skill/main`.

- **Claude Code**: `~/.claude/skills/human-narrative/SKILL.md` for every project, or `.claude/skills/human-narrative/SKILL.md` for one.
- **Codex CLI**: `~/.codex/skills/human-narrative/SKILL.md`, or `.agents/skills/human-narrative/SKILL.md` for one project.
- **opencode**: `~/.config/opencode/skills/human-narrative/SKILL.md` or `.opencode/skills/human-narrative/SKILL.md`. opencode also reads the Claude Code and Codex paths.
- **Gemini CLI**: `~/.gemini/skills/human-narrative/SKILL.md`.
- **Cursor**: the body of `SKILL.md` as `.cursor/rules/human-narrative.mdc`, with `alwaysApply: true` in the frontmatter.
- **Claude.ai and Claude Desktop**: zip a folder named `human-narrative` containing `SKILL.md` and upload it under Skills in settings.
- **Anything else**: paste the body of `SKILL.md`, without its frontmatter, into `AGENTS.md`.

## Whole-session version for Claude Code

The skill applies when the agent decides the task calls for it. The output style applies to everything, reasoning included. Save `output-style.md` to `~/.claude/output-styles/human-narrative.md` and run `/output-style human-narrative`.

## Sources

The rules are adapted from work in three areas: what separates model text from human text, how readers process a sentence, and which phrases recur in model output.

**What separates model text from human text**

- [StoryScope](https://arxiv.org/abs/2604.03136) (Russell, Rajendhran, Pham, Iyyer, and Wieting, COLM 2026) told AI fiction from human fiction by discourse-level choices such as tidy plots, spelled-out themes, and less structural variety, without relying on word choice. The chaining rules and the ban on stated morals come from here.
- [Terčon and Dobrovoljc 2025](https://arxiv.org/abs/2510.05136) surveyed the lexical, syntactic, discourse, and stylistic patterns reported for machine-generated text. The rules against hedge words on known facts and against symmetric paragraph structure draw on it.
- [Przystalski, Argasiński, Grabska-Gradzińska, and Ochab](https://arxiv.org/abs/2507.00838) (Expert Systems with Applications, 2026) separated human and model text on ten-sentence samples. That result is why the rules apply to short text like commit messages, not only to long documents.
- [Tarım and Onan 2025](https://arxiv.org/abs/2507.10475) used burstiness, the variation in sentence length, as one of several stylometric measures separating model text from human text. [Rodrigues, Sturm, and Pinheiro 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/), in iScience, found human texts vary more in length while model text runs more formal, structured, and upbeat. The rule to vary sentence length follows from both.
- [DetectGPT](https://arxiv.org/abs/2301.11305) (Mitchell, Lee, Khazatsky, Manning, and Finn, 2023), [Fast-DetectGPT](https://arxiv.org/abs/2310.05130) (Bao, Zhao, Teng, Yang, and Zhang, 2023), and [Binoculars](https://arxiv.org/abs/2401.12070) (Hans et al., 2024) detect model text from how predictable each token is given the ones before it. The concreteness and word-choice rules push against that signal.

**How readers process a sentence**

- [Gopen and Swan 1990](https://www.americanscientist.org/blog/the-long-view/the-science-of-scientific-writing), in American Scientist, described where readers expect old and new information to sit: the topic at the start of a sentence, the stress at the end. Joseph Williams's *Style: Lessons in Clarity and Grace* extends the idea to topic strings across a paragraph. The chain-by-topic rule and the paragraph-break rule come from these two.

**Which phrases recur in model output**

- [DecEptioner's list of common AI phrases](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses) supplied the ban list.
