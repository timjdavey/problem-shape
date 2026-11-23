---
type: Experiment
title: Do the axes predict where AI agents fail?
description: A scratch proof of concept from October 2026 that tagged METR's public tasks on the 40 axes of the time. AI models designed and ran it, and no person has reviewed it in depth yet.
---
# Do the axes predict where AI agents fail?

> **Scratch proof of concept. Read this first.** Claude Fable designed this pilot and Claude Opus built and ran it, each working on its own with no person checking the steps. The tags came from Claude Opus and Sonnet subagents. No person has reviewed the design, the code or the results in depth. Don't put much trust in any number here, and don't spend long reviewing it yet. The project is at the idea stage. A more detailed and comprehensive review is planned soon, and this page will change when it lands.

In March 2025 METR timed people doing about 170 software and research tasks. Then they ran AI agents on the same tasks. Longer tasks were harder for agents, which is no surprise. But METR also scored each task on 16 "messiness" factors, like whether the agent could check its own answer. After allowing for length, each extra point of messiness cut agents' mean success by roughly 8.1%, in METR's words.

That result is the README's best evidence that tagging problems pays off. But it is METR's evidence, not ours. This experiment asks whether our axes do the same job. A model tags each task on every axis without seeing how agents did. Then we check which axes separate the tasks agents pass from the ones they fail. The answer should show which six to ten axes do real work, and which ones are decoration on this kind of task.

## What is public

The run data is public: about 24,000 agent runs on 228 tasks by 20 models, with human times. Most of the task text is not. METR keeps it private so models aren't trained on it. What is public is the full text for 11 task families, and a one or two sentence summary of every family in the [HCAST paper](https://arxiv.org/abs/2503.17354). The pilot tags those. METR's own messiness labels aren't public either.

So the pilot has 86 things to tag: 23 full task prompts and 63 family summaries. Families share setups, so the real sample size is about 74, one per family. That is only enough to find large effects.

To run the full test, ask METR for the task text and their messiness labels. Put each task's text in `data/metr_task_text/<family>__<task>.txt` and their labels in `data/metr_messiness.csv`. `fetch.py` and `analyse.py` pick both up without any changes.

**Never commit anything in `data/`.** It holds task text that METR asked to keep out of training data. It is gitignored.

## How to run it

You need [uv](https://docs.astral.sh/uv/) and an Anthropic API key. Each script lists its own dependencies, so there's nothing to install.

1. `uv run fetch.py` downloads the runs and the public task text into `data/`.
2. Edit and commit `predictions.md` before going further.
3. `uv run tag.py --limit 2` is a smoke test. Then `uv run tag.py` tags every unit three times. It costs roughly $20 to $40 and can be stopped and restarted.
   To tag without an API key, run `uv run blind.py prepare` instead. It writes batches to `data/blind/<pass>/batch_<n>/`, each with a rubric, the tasks under anonymous ids and an `INSTRUCTIONS.md`. Point a subagent or a fresh chat at each folder. Then run `uv run blind.py collect` and `uv run tag.py --collect-only`.
4. Hand tag the 15 tasks in `data/hand_tags_tasks.md`, using `data/hand_tags_template.csv`. Save it as `data/hand_tags.csv`.
5. `uv run analyse.py` writes `results.md`.

To check the archived numbers you only need steps 1 and 5. `analyse.py` reads the tags in `tags.csv`, so it needs no API key and gives the same results every time. Step 3 is different. `tag.py` builds its rubric from today's axis files, so tagging again would score the 50 current axes and not the 40 used here.

## What each step tests

The tagger sees only the task text. It never sees the task's name, its human time or any result. Three passes act as three raters: Opus, Sonnet and Opus again with the axes in a shuffled order. Your hand tags are a fourth.

`analyse.py` then works through six steps.

0. **Baseline.** It fits METR's curve of success against task length for each model, using the public runs alone. If the time horizons don't match METR's, the data is wrong. They do: Claude 3.7 Sonnet comes out at 62 minutes and Claude Opus 4.5 at 292.
1. **Reliability.** If three raters can't agree on an axis, that axis can't predict anything. We measure agreement with a statistic called Krippendorff's alpha and drop axes below 0.4. Axes that barely vary across these tasks are marked untested, not failed.
2. **Tagger check.** The same pipeline also tags METR's 16 factors. If our tags don't reproduce METR's slope of about −0.08, the tagger is broken and the rest means nothing.
3. **One axis at a time.** For each axis, how much worse do agents do at the hard end than at the easy end, beyond what length predicts? Testing 40 axes at once throws up false hits, so p values are corrected with the Benjamini Hochberg method.
4. **The shortlist.** All axes go into one model together. Any one fit will pick up some noise, so we refit on hundreds of random halves of the families and keep only the axes that survive almost every time. This is called stability selection.
5. **Added value.** Do our axes predict better than METR's messiness sum? Each family is held out in turn and predicted from the rest. If the axes add nothing, they are METR's factors in new words.
6. **Shuffle test.** Give each family another family's tags, refit and repeat a thousand times. Real tags have to beat shuffled ones.

## Does the analysis work?

Before any real tags existed, Claude Opus tested `analyse.py` twice on fake tags. In the first run one axis was planted to track agent success and the other 39 were random. The planted axis was the only one on the shortlist, and the shuffle test passed with p = 0.01. In the second run every axis was random. The shortlist came back empty, no axis survived the correction and the shuffle test failed with p = 0.35. So the method finds a real signal and doesn't make one up.

## What the pilot found

The pilot was tagged by nine Claude subagents in October 2026, three for each pass. Each saw only the rubric and anonymised task text. Sonnet was blocked by a safety classifier on six hacking challenge tasks, so those have two raters instead of three. The full numbers are in [results.md](results.md).

In short, the pilot did not show that our axes predict agent success.

| Question | Short answer |
|---|---|
| Can a model tag tasks well enough to trust? | Yes on full task text, no on one line summaries. |
| Do our axes predict success beyond task length? | Not on this data. One axis gave a lead that didn't survive correction, and real tags did no better than shuffled ones (p = 0.77). |
| How many axes could these tasks test at all? | 25 of the 40 we had then. |

**The tagger works, but only on real task text.** On the 23 tasks with their full prompt, our tags of METR's 16 factors give a slope of −0.085 per point (95% interval −0.15 to −0.02). METR's own human labels gave −0.081. On the 139 tasks tagged from a one line family summary, the slope falls to −0.015 and the interval spans zero. So the method is sound and the summaries are too thin to tag. By the rule we set in advance, nothing below the tagger check counts as evidence until we have the full text.

**One axis stands out, as predicted.** Easy vs hard to verify was the top guess in `predictions.md`. It is the strongest single axis: agents succeed about 23 points less often on hard to verify tasks than on easy ones of the same length. It is also the only axis to make the shortlist. METR's closest factor, "no provided verification mechanisms", is also their strongest. But the result doesn't survive correction for testing 25 axes (q = 0.12), and nothing predicts better than chance on held out families. Treat it as a lead, not a finding.

**Fifteen axes have nothing to grip on in these tasks.** Eight were mostly tagged n/a, among them thin vs fat tails and additive vs multiplicative. Five barely varied, like survivable vs ruinous. Two the taggers couldn't agree on, small vs large world and slack vs time pressure. Skin in the game shows the problem best: one tagger called it hard everywhere, another n/a everywhere and a third mixed. That says the axis is unclear when the solver is an AI agent, which is worth fixing in the axis file whatever happens next.

## What a proper test needs

1. **A review by a person.** Nobody has yet checked the design, the code or the tags in depth, and that comes before anything else.
2. **The full text of every task, with results for each one.** One line summaries are too thin to tag.
3. **Far more than 11 task families.** A sample this small can only show very large effects.
4. **Benchmarks picked for their spread across the axes.** More lab tasks of the same kind would leave the same 15 axes untested.
5. **Tags from people as well as models.**
6. **Predictions committed before any task is tagged.**

If you hold task text and per task results and would like to run this with us, please get in touch. The tagging can run on your side, so the text never leaves your hands.

## Old axis names

The tags and results use the axis names from when the pilot ran, and the slugs in `tags.csv` still do. Of the 40 axes, 19 kept their names and 21 changed. The [update log](../../log.md) gives the reason for each rename.

| Name in the pilot | Page now |
|---|---|
| Additive vs multiplicative | [Flat vs compounding results](../../axes/flat-vs-compounding-results.md) |
| Choose vs design the options | [Given vs invented options](../../axes/given-vs-invented-options.md) |
| Clustered vs scattered | [Clustered vs scattered runs](../../axes/clustered-vs-scattered-runs.md) |
| Coherent vs incoherent | [Coherent vs incoherent runs](../../axes/coherent-vs-incoherent-runs.md) |
| Complete vs incomplete score | [Direct vs proxy measure](../../axes/direct-vs-proxy-measure.md) |
| Decomposable vs tangled | [Reductionist vs holistic](../../axes/reductionist-vs-holistic.md) |
| Deterministic vs statistical | [Deterministic vs stochastic](../../axes/deterministic-vs-stochastic.md) |
| Easy vs hard to verify | [Verifiable vs unverifiable](../../axes/verifiable-vs-unverifiable.md) |
| Even vs lopsided errors | [Equal vs unequal error costs](../../axes/equal-vs-unequal-error-costs.md) |
| Kind vs wicked feedback | [Immediate vs delayed feedback](../../axes/immediate-vs-delayed-feedback.md) |
| Known vs unknown odds | [Risk vs Knightian uncertainty](../../axes/risk-vs-knightian-uncertainty.md) |
| Long vs short predictability horizon | [Stable vs chaotic](../../axes/stable-vs-chaotic.md) |
| Loose vs tight coupling | [Buffered vs cascading](../../axes/buffered-vs-cascading.md) |
| Single vs multiple objectives | [Single objective vs trade-offs](../../axes/single-objective-vs-trade-offs.md) |
| Slack vs time pressure | [Ample time vs time pressure](../../axes/ample-time-vs-time-pressure.md) |
| Small vs large world | [Foreseeable vs unforeseeable outcomes](../../axes/foreseeable-vs-unforeseeable-outcomes.md) |
| Smooth vs rugged | [Forgiving vs exacting](../../axes/forgiving-vs-exacting.md) |
| Spread vs single bet | [Diversifiable vs single bet](../../axes/diversifiable-vs-single-bet.md) |
| Stationary vs drifting | [Durable vs perishable lessons](../../axes/durable-vs-perishable-lessons.md) |
| Test vs watch | [Interventional vs observational](../../axes/interventional-vs-observational.md) |
| Tractable vs intractable search | [Tractable vs intractable options](../../axes/tractable-vs-intractable-options.md) |

Two of these changed in meaning as well as in name. Kind vs wicked feedback was a bundle of several ways experience can mislead you, and only the delay stayed on the page it became. Convergent vs divergent kept its name but was reworded to pull it apart from Single vs multiple objectives, which the taggers had put in the same band on 167 of 252 units.

## Limits

- About 74 families can only show large effects. A null result for one axis is weak evidence.
- In the pilot most tasks are tagged from a one or two sentence summary. Axes that need detail, like reversible vs irreversible, will be noisy until METR shares the full text.
- These are lab tasks for agents, so they cover a narrow band of problems. Each task goes to one agent working alone in a sandbox with a time limit. Nobody else has a say in the goal, nobody answers for the result and no real loss is at stake. So the tasks sit at the easy end of the axes about values, accountability and stakes, and those axes are untested here, not refuted.
- The match of −0.085 with −0.081 is a loose one. It rests on 23 tasks from 11 families. METR's labels came from people and ours from Claude models. METR's paper also gives its measure once as a ratio (Section 4.1) and once as a plain difference (Figure 10), and ours is the difference.
- The git history can't prove the guesses came first. [predictions.md](predictions.md), the tags and the results all entered the repo in one commit, so "as predicted" is our word and not a registered prediction.
- The tagger is a model tagging tasks for models. If it shares the agents' blind spots, tags and failures could line up for reasons that have nothing to do with the axis. The hand tags are the guard against this.
- METR say their own factors were picked partly because they seemed to hurt agents. Our axes weren't, which is a fairer test but means a smaller effect is still a good result.

## History of this folder

The pilot ran in October 2026 on the 40 axes the project had then. There are 50 now, and 21 of the 40 have new names, listed under [Old axis names](#old-axis-names). The folder was deleted on 22 October 2026, because the test couldn't be run properly without METR's private task text. It came back on 23 November 2026 so that the work is open to check.

On the same day Claude Fable 5.2 reran `fetch.py` and `analyse.py` against METR's public data, and every number in [results.md](results.md) came out the same. That shows the scripts give the same output from the same tags. It is not a review of the design. Claude Fable 5.2 also corrected this page, by fixing the count of untestable axes, adding the map of old names and adding two of the limits above.
