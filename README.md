# human-narrative

Text written by an agent costs more to read than it needs to. This skill changes how the agent writes anything a person will read, so the information arrives in the order the reader needs it and each sentence follows from the last. The result is noticeably easier to follow.

```bash
npx skills add sushruth/human-narrative-skill
```

The command installs the skill into every coding agent it finds on the machine. `-g` installs under your home folder; `-a claude-code` picks one agent. To remove it, delete the `human-narrative` folder it created. For an agent the CLI doesn't know, the whole skill is the one file, `SKILL.md`.

## What changes

Replies, docs, PR descriptions, code comments, and commit messages come out in a different shape.

- The first line says where the reader is. What they came for is next. Explanation comes after, if at all.
- Facts are stated flat. Judgments carry a size word. "Verified" appears only when something was run.
- Nothing ends on a moral or a summary, and open questions are left open.
- The usual filler words are banned, along with the mid-sentence run of examples between em-dashes.
- Sections answer the reader's questions. A benefits section, a "do X, not Y" sentence, and a line saying what kind of document this is all go, and the asker's hedges and shorthand are translated for the reader.

Code is left alone. The rules apply to prose, and the Claude Code output style keeps the coding instructions in place.

The skill is not built to pass AI detectors. Several of its sources are detector papers, used here to describe what model text does.

The skill file is about 17 KB. The agent carries all of it in context while the skill is active.

## Side by side

This is the clearest of 30 pairs I read, one run each on Claude Code (Sonnet), same prompt, neither output edited. Most pairs changed less, and a few came out worse; [example-runs.md](example-runs.md) has all of them.

**Prompt:** Write the opening of the README for `tzsplit`, a CLI that reads a CSV of timestamps and splits it into one file per local calendar day, using each row's own timezone column. Two or three sentences, then the install command (`cargo install tzsplit`).

Without the skill:

`````
```markdown
# tzsplit

`tzsplit` reads a CSV of timestamps and splits it into one file per local calendar day. Each row is bucketed using its own timezone column, so a row stamped 23:30 in `America/Los_Angeles` lands on that Pacific day, not the UTC one.

    cargo install tzsplit
```

I didn't state the timezone column's format (IANA names vs. offsets), the output filenames, or the flags, because I don't know them. Add those once they're settled.
`````

With the skill:

`````
```markdown
# tzsplit

Each row in a multi-timezone CSV reaches midnight at a different moment. `tzsplit` reads a CSV of timestamps and writes one file per local calendar day, using each row's own timezone column to decide which day it belongs to.

    cargo install tzsplit
```
`````

The first opens with what the tool does and ends with a note about what the writer left out. The second opens with the situation the reader is already in, then says what the tool does about it.

## Whole-session version for Claude Code

The skill applies when the agent decides the task calls for it. The output style applies to everything, reasoning included. Save `output-style.md` to `~/.claude/output-styles/human-narrative.md` and run `/output-style human-narrative`.

## Sources

The rules are adapted from work in three areas: what separates model text from human text, how readers process a sentence, and which phrases recur in model output.

**What separates model text from human text**

- [StoryScope](https://arxiv.org/abs/2604.03136) (Russell, Rajendhran, Pham, Iyyer, and Wieting, COLM 2026) told AI fiction from human fiction by discourse-level choices such as tidy plots, spelled-out themes, and less structural variety, without relying on word choice. The chaining rules and the ban on stated morals come from here.
- [Terčon and Dobrovoljc 2025](https://arxiv.org/abs/2510.05136) surveyed the lexical, syntactic, discourse, and stylistic patterns reported for machine-generated text. The rules against hedge words on known facts and against symmetric paragraph structure draw on it.
- [Przystalski, Argasiński, Grabska-Gradzińska, and Ochab](https://arxiv.org/abs/2507.00838) (Expert Systems with Applications, 2026) separated human and model text on ten-sentence samples. That result is why the rules apply to text as short as a commit message.
- [Tarım and Onan 2025](https://arxiv.org/abs/2507.10475) used burstiness, the variation in sentence length, as one of several stylometric measures separating model text from human text. [Rodrigues, Sturm, and Pinheiro 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12969083/), in iScience, found human texts vary more in length while model text runs more formal, structured, and upbeat. The rule to vary sentence length follows from both.
- [DetectGPT](https://arxiv.org/abs/2301.11305) (Mitchell, Lee, Khazatsky, Manning, and Finn, 2023), [Fast-DetectGPT](https://arxiv.org/abs/2310.05130) (Bao, Zhao, Teng, Yang, and Zhang, 2023), and [Binoculars](https://arxiv.org/abs/2401.12070) (Hans et al., 2024) detect model text from how predictable each token is given the ones before it. The concreteness and word-choice rules push against that signal.
- [Russell, Karpinska, and Iyyer](https://arxiv.org/abs/2501.15654) (ACL 2025) had five annotators who use LLMs often vote on 300 professionally edited US nonfiction articles; the majority vote got 299 right, including paraphrased and humanized text. Their explanations named vocabulary in 53.1% of cases and sentence structure in 35.9%. The skill works on both, and it does not claim to hide model origin from a reader like this.
- [Reinhart and co-authors](https://arxiv.org/abs/2410.16107) (arXiv, v2) had GPT-4o and Llama 3 continue human texts and counted grammatical features. GPT-4o used present participial clauses at 5.3 times the human rate and nominalizations at 2.1 times. The preference for precise verbs over abstract nouns draws on it.

**How readers process a sentence**

- [Gopen and Swan 1990](https://www.americanscientist.org/blog/the-long-view/the-science-of-scientific-writing), in American Scientist, described where readers expect old and new information to sit: the topic at the start of a sentence, the stress at the end. Joseph Williams's *Style: Lessons in Clarity and Grace* extends the idea to topic strings across a paragraph. The chain-by-topic rule and the paragraph-break rule come from these two.
- [Mayo, Schul, and Burnstein 2004](https://doi.org/10.1016/j.jesp.2003.07.008), in the Journal of Experimental Social Psychology, compared two accounts of how a negated statement is stored: fused into one meaning, or as the core claim plus a separate negation tag. When no ready schema matches the negation's meaning, the negated claim produced associations that run against the negation. That is the reason the negation rules cut the negated half unless it is a fact the reader acts on. The study tests negated statements in general, not model text.

**Which phrases recur in model output**

- [DecEptioner's list of common AI phrases](https://deceptioner.site/blog/what-are-common-phrases-that-ai-uses) supplied the ban list.
- [Kobak, González-Márquez, Horvát, and Lause](https://arxiv.org/abs/2406.07016) (arXiv, v5, July 2025) compared 2024 word frequencies in 15.1 million PubMed abstracts against a trend extrapolated from 2021 and 2022. "Delves" appeared 28.0 times as often as expected and "underscores" 13.8 times. The data is biomedical abstracts only, so it supports the ban list for that genre alone.
