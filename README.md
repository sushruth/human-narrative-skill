# human-narrative

Coding agents describe their own changes the way a brochure would. This skill hands them a different set of writing rules.

```bash
npx skills add sushruth/human-narrative-skill
```

The command finds the coding agents installed on the machine and copies `SKILL.md` into each one's skill folder. Add `-g` to install for the user instead of the current project, or `-a claude-code -a codex` to name the agents yourself.

## What the rules cover

Once loaded, the skill applies to anything a person will read: PR descriptions, commit messages, code comments, docs, and chat replies. The file is about 15 KB and the agent takes all of it into context whenever the skill is active. The rules are about order and size more than about vocabulary.

- **Order.** The first line names the reader's situation. The thing they came for comes second. Mechanism, examples, and caveats come after, if at all.
- **Size.** Facts are stated flat, with no qualifier. Judgments carry a size word. "Verified" is reserved for checks that were actually run.
- **Chaining.** Each sentence opens on something the previous sentence put down. Paragraphs break where the reader's question changes, so they come out uneven.
- **Endings.** No moral, no closing summary. Open questions stay open.
- **Ban list.** The usual filler words, and the mid-sentence run of examples set off by em-dashes or parentheses.

## Install by hand

Both files are at `https://raw.githubusercontent.com/sushruth/human-narrative-skill/main`.

- **Claude Code**: save `SKILL.md` to `~/.claude/skills/human-narrative/SKILL.md` for every project, or to `.claude/skills/human-narrative/SKILL.md` for one.
- **Claude Code output style**: save `output-style.md` to `~/.claude/output-styles/human-narrative.md` and run `/output-style human-narrative`. The style governs the whole session, reasoning included, and keeps Claude Code's own coding instructions in place.
- **Codex CLI**: save `SKILL.md` to `~/.codex/skills/human-narrative/SKILL.md` for every project, or to `.agents/skills/human-narrative/SKILL.md` for one.
- **opencode**: save `SKILL.md` to `~/.config/opencode/skills/human-narrative/SKILL.md` or `.opencode/skills/human-narrative/SKILL.md`. opencode also reads the Claude Code and Codex paths above.
- **Gemini CLI**: save `SKILL.md` to `~/.gemini/skills/human-narrative/SKILL.md`.
- **Cursor**: save the body of `SKILL.md` as `.cursor/rules/human-narrative.mdc` with `alwaysApply: true` in the frontmatter.
- **Claude.ai and Claude Desktop**: zip a folder named `human-narrative` containing `SKILL.md` and upload it under Skills in settings.
- **Anything else**: paste the body of `SKILL.md`, without its frontmatter, into `AGENTS.md`.

## Before and after

One prompt to Claude Code running Sonnet, one run with the skill and one without:

> In three or four sentences, explain to a new teammate why the team uses feature flags instead of long-lived feature branches.

| Without the skill | With the skill |
| :--- | :--- |
| Feature flags let you merge small changes to main continuously, so code stays integrated and tested instead of drifting apart in a branch for weeks. Long-lived branches build up merge conflicts that get worse the longer they live, and they hide bugs until one big scary merge at the end. Flags also let you ship code dark, then turn features on for specific users or roll back instantly without a redeploy — a branch merge has no equivalent "undo." Net effect: smaller diffs, faster feedback, safer rollout. | Long-lived feature branches drift from main for weeks, so merging one back means untangling conflicts against everything else that shipped in the meantime. Feature flags skip that: the code merges to main right away, sits behind a flag, and turns on for real users once it's ready — no big-bang merge, no stale branch. That also means you can ship a half-finished feature dark and test it in production before flipping it on, and roll it back by flipping the flag instead of reverting a merge. The tradeoff is you carry flag-check code and have to remember to delete flags once a feature is fully rolled out — branches don't leave that residue, but they cost more at merge time. |

Without the skill the reply closes on a slogan. With the skill each sentence picks up where the last one stopped, and the final sentence is the tradeoff. The em-dashes survived in both.

## What changed, and what didn't

One run per prompt in Claude Code, Codex CLI, opencode, and Gemini CLI, each with and without the skill loaded. Single runs, so read these as observations rather than measurements.

- Explanations a paragraph or longer lose their sub-headers and hedge words, and end on a tradeoff instead of a summary.
- Text under three sentences looks the same either way. Commit subjects, one-line PR descriptions, and short chat replies all fell in that bucket.
- Codex and Gemini stop stringing examples between em-dashes. Sonnet and DeepSeek keep doing it with the rule in front of them.

The skill closes some of the gap between agent prose and a careful person's prose, not all of it.

## Sources

The rules borrow from the following.

- **Where the difference lives.** [StoryScope](https://arxiv.org/abs/2604.03136) (Russell, Rajendhran, Pham, Iyyer, and Wieting, 2026) told AI fiction from human fiction by discourse-level choices such as tidy plots, spelled-out themes, and less structural variety, without leaning on word choice. The chaining rules and the ban on stated morals come from here.
- **Sentence rhythm.** [Tarım and Onan 2025](https://arxiv.org/abs/2507.10475) used burstiness, the variation in sentence length, as one of several stylometric measures that separate model text from human text. [Rodrigues, Sturm, and Pinheiro 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/), in iScience, found that human texts vary more in length while model text runs more formal, structured, and upbeat. The rule to vary sentence length follows from both.
- **Predictability.** [DetectGPT](https://arxiv.org/abs/2301.11305) (Mitchell, Lee, Khazatsky, Manning, and Finn, 2023), [Fast-DetectGPT](https://arxiv.org/abs/2310.05130) (Bao, Zhao, Teng, Yang, and Zhang, 2023), and [Binoculars](https://arxiv.org/abs/2401.12070) (Hans et al., 2024) detect model text from how predictable each token is given the ones before it. The concreteness and word-choice rules push against that signal.
- **Hedging and repetition.** [Terčon and Dobrovoljc 2025](https://arxiv.org/abs/2510.05136) surveyed the lexical, syntactic, discourse, and stylistic patterns reported for machine-generated text. The rules against hedge words on known facts and against symmetric paragraph structure draw on that survey.
- **Short samples.** [Przystalski, Argasiński, Grabska-Gradzińska, and Ochab](https://arxiv.org/abs/2507.00838), in Expert Systems with Applications 2026, separated human and model text on ten-sentence samples using stylometric features. That result is why the rules apply to commit messages and comments and not only to long documents.
- **Information order.** [Gopen and Swan 1990](https://www.americanscientist.org/blog/the-long-view/the-science-of-scientific-writing), in American Scientist, described where readers expect old and new information to sit in a sentence: the topic at the start, the stress at the end. Joseph Williams's *Style: Lessons in Clarity and Grace* extends the idea to topic strings across a paragraph. The chain-by-topic rule and the paragraph-break rule come from these two.
- **Filler phrases.** [DecEptioner's list of common AI phrases](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses) supplied the ban list.
