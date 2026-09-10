# Maintaining human-narrative

You are about to change a rule set whose only test is running it. Below is what earlier sessions learned doing that, in the order you'll need it.

## What's here

- `SKILL.md` is the rules. Its frontmatter `description` decides when an agent loads the skill, so a change in scope means a change there too.
- `output-style.md` is the same body under Claude Code output-style frontmatter, with `keep-coding-instructions: true` and an intro sentence that adds reasoning and live conversation to the scope.
- `README.md` is written under the rules and is their first test case.

There is no build, no test suite, and no CI. Work lands on `main`.

## Changing a rule

1. Start from a failure in real output, not from a principle. Name the model and quote the text that went wrong. Every rule in the file traces back to a trial like that.
2. Write the rule as something checkable: a cap, a position on the page, or a bad-to-good pair. "One sentence under fifteen words," "the command is the second thing on the page," and "eliminates flaky tests becomes catches most of the flaky tests" all moved models. Abstract statements of the same idea did not.
3. Add a matching item to Before sending. The checklist is the part that shapes a draft. A rule stated once in the body and never checked was how a README opener with no direction passed every check until check 6 existed.
4. If the rule touches the opener, put a one-line version in the intro paragraph as well. In the opener trials, the section rule alone left opencode/DeepSeek describing the object. The one-liner in the intro flipped it.
5. Look for the failure the new rule can cause. The first paragraph-break rule keyed on grammatical subject and merged single-actor explainers into walls on Sonnet and DeepSeek. Keying on the reader's question, with a five-to-six sentence cap, undid that.
6. Don't prescribe surface markers of voice, and don't measure the output against a person's writing. Rules asking for "Honestly," and "That said," were cut because the real failure was structural. "Not an AI essay" and "like a careful person wrote it" were both removed for setting a bar the text can't be checked against. Describe the reader and the text instead.
7. Keep worked examples in the rule out of the trial artifact's domain. The opener rule's example is a photo-dedup CLI because the trial artifact was this repo's README; an example about writing rules would have handed the model the answer and the round would have measured copying.
8. Check the under-claiming side. Every existing check catches over-claiming, so the path of least resistance is a sentence that names a change and not its direction. Check 6 covers that; keep it when you reorder the list.

## Running a trial

- Use a fixed artifact. Past rounds used this repo's README generated from a fact sheet, and the prompts now in the README example. Same prompt, one run per condition, with and without the current `SKILL.md` loaded. Change only the skill between rounds.
- Run on more than one model. Past rounds used Claude Code on Sonnet, Codex CLI, opencode on DeepSeek, and Gemini CLI through `agy`. A rule that flips two of four is worth keeping. Sonnet has opened on a habit list with an em-dash in every opener trial so far.
- When the CLIs aren't available, a subagent that reads the rules file and has no other tools is an acceptable stand-in. It is still one run per condition.
- Score each output against a short list written before the run, one criterion per rule under test. The README rounds used four: first line is a situation, not the object; reception claims carry a size word and no hero verb; the install command sits right under the first line; no em-dash list in the opener. Three of four criteria is a pass for an output, three of four models is a pass for the round.
- Stop when a round moves nothing. Three rounds was enough for the opener: round one moved two models, round two moved none, round three moved one. A model that fails the same criterion every round is not going to be reached by more text in the file.
- Put the rounds in the commit body: what changed in the skill, which models flipped, which didn't. The git log is the lab notebook, and it is the only place the trial results live.

### Invoking the CLIs

Run every CLI with its working directory outside this repo, or the repo's own config leaks into the run. Each one takes the rules differently, and each has a flag that bit once:

- Claude Code: `claude -p --model sonnet --disable-slash-commands --setting-sources user --append-system-prompt-file SKILL.md "<prompt>"`. The rules go in as a system prompt; the other three get them inlined above the prompt text.
- Codex: `codex exec --ignore-user-config --skip-git-repo-check -C <dir> -o <file> "<rules + prompt>"`. `--ignore-user-config` also changes which model and reasoning effort you get.
- opencode: `opencode run --pure -m deepseek/deepseek-v4-flash "<rules + prompt>"`. `--pure` does not fully isolate; a private repo name leaked into one baseline run.
- Gemini: `agy --disable-slash-commands -p="<rules + prompt>"`. With other flags present, `-p <prompt>` takes the next flag as the prompt; the `=` form is required.

Inlined rules must not start with the file's `---` frontmatter. Codex, opencode, and agy all parse a leading `---` as a flag and exit. Strip the frontmatter and lead with a sentence such as "Follow these writing rules for your reply."

## Keeping the two files in sync

`output-style.md` once drifted several rules behind. After any edit to either file, diff the bodies:

```bash
diff <(awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' SKILL.md) \
     <(awk 'BEGIN{n=0} /^---$/{n++; next} n>=2' output-style.md)
```

The only expected difference is the first sentence of the intro. Frontmatter differs by design and is excluded from the diff.

The README states the skill file's size in KB. Update the number when an edit moves it across a boundary.

## Installed copies

Editing the repo changes nothing an agent reads. The copies under `~/.claude/skills/human-narrative/`, `~/.agents/skills/human-narrative/`, and `~/.claude/output-styles/human-narrative.md` went a full day stale once, and the output style had been stale for three weeks. After a change lands, copy the two files over those paths or re-run `npx skills add sushruth/human-narrative-skill`, then `cmp` each pair.

## The README

- It has to pass Before sending. The opener is the reader's situation in one short sentence, then which way the text moves and how far with a size word, then the install command. Nothing about what the skill is until after that.
- One side-by-side example, from real runs, unedited. Say the model and that it was one run each. If a harness footer was stripped, say so. Regenerate the example when the "with" sample breaks a rule the set has since gained; that has happened once already, over an em-dash list and an invented time estimate.
- Keep every citation. Reword an entry only against the paper's title, authors, venue, and finding. When the network blocks a fetch, verify by search rather than dropping a detail, and name in the commit any source you couldn't reach. Two entries have been corrected this way: Tarım and Onan compare diffusion and autoregressive text, and Rodrigues, Sturm, and Pinheiro measure length variation, not syntactic pacing.
- Install paths change under you. Check current agent docs before touching them. There is no install-by-hand list; the CLI covers the agents, and `SKILL.md` is the whole skill for anything else.
- A README generated by an agent gets a line-by-line fact check by a person before it lands. The last one needed four fixes, including a CLI listed as tested that wasn't. List the fixes in the commit body.
- Don't name the session, tool, or model that wrote it.

## Commits

Imperative subject. The body says what changed and why. Rule changes carry the trial rounds. Small wording fixes can go without a body.
