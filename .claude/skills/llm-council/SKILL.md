---
name: llm-council
description: Run a question past a council of independent Claude subagents (Karpathy's LLM Council method) - parallel answers, anonymous peer ranking, chairman synthesis. Use when the user says "council", "/llm-council", or wants a pressure-tested answer to a hard design, pitch or strategy question.
---

# LLM Council

Karpathy's [llm-council](https://github.com/karpathy/llm-council) method, run on Claude subagents instead of OpenRouter, so it costs nothing extra (no API keys).

## Council
Default: 4 members, each a separate subagent (Agent tool, run in parallel), each with a different `model` and lens so answers actually differ:

| Member | model | Lens |
|---|---|---|
| A | opus | First principles: what is actually true, what is the strongest answer |
| B | sonnet | Pragmatist: what ships in the time and budget we have |
| C | haiku | Skeptic: what breaks, what judges/users would attack |
| D | sonnet | User/judge view: what is clearest and most convincing |

The user may name a different size, models or lenses; use theirs. If a model is unavailable, fall back to the session model and keep the lens.
Give every member the same full context: the question, plus relevant files/docs (for this repo: `STATUS.md`, the relevant `docs/` files). Members must not see each other.

## Stage 1: First opinions
Each member answers the question independently in <=300 words, ending with a one-line bottom line. Run all members in one message (parallel).

## Stage 2: Anonymous peer review
1. Shuffle the Stage 1 answers and relabel them **Response A, B, C, D**. Remove anything that reveals the author (model name, lens name). Keep the label -> member map to yourself.
2. Send every member all anonymized responses (including its own, unlabeled as such). Each one:
   - critiques each response in 1-3 lines (accuracy, insight, gaps),
   - ends with exactly:
     ```
     FINAL RANKING:
     1. Response X
     2. Response Y
     ...
     ```
3. Parse each FINAL RANKING block. Aggregate: average rank position per response (lower = better). If a block won't parse, drop that reviewer and say so.

## Stage 3: Chairman
One more subagent (model: opus) gets the question, all de-anonymized Stage 1 answers, all Stage 2 reviews and the aggregate ranking. It writes the final answer: the best synthesis, not a vote count. It states where the council disagreed and which side it took, and why.

## Output to the user
1. **Council answer** (the chairman's synthesis): lead with it.
2. **Leaderboard**: table of member/model/lens, average rank.
3. **Key disagreements**: 1-3 bullets.
4. Offer the full Stage 1 answers and Stage 2 reviews on request (don't dump them by default).

If the result is a decision for this repo, offer to log it in `docs/03-design/decisions.md`.
