# The council pattern — running several models on one hard question

[香港廣東話](pattern_zhHK.md)

A council is a small, structured way to get more than one AI model's opinion on a question,
and to combine their answers into something better than any single one, deliberately, not
just by averaging them together.

## When it's worth the extra cost

A council earns its cost on **judgment calls without a checkable answer** — the kind of
question where two informed, competent people could reasonably land in different places,
and there is no ground truth to check the answer against directly. Examples: a go/no-go
call under real uncertainty, a methodology critique, a piece of writing where "good" is a
judgment call, a strategic trade-off with no single right answer.

It is **not** worth it for questions with a checkable answer: a bug either is or isn't in
the code; a fact is either right or wrong. For those, one strong model plus a fresh,
independent check (a second pass that verifies the claim directly, not just re-reads the
same reasoning) is cheaper and just as reliable. Spending a council on a lookup problem
buys nothing but cost.

## The pattern: parallel answers, then synthesis, not averaging

The naive approach to combining several models is to blend their outputs, or have each
one see and build on the previous one's answer, layer by layer. Both are used in practice
and both share the same weakness: nothing in the process actively identifies and discounts
a bad answer. A weak or wrong contribution just gets folded into the average, quietly
pulling the result down.

The pattern that avoids this has three stages:

1. **Parallel, independent answers.** Every model gets the exact same brief, at the same
   time, with no visibility into what any other model is doing. This is what makes
   convergence meaningful: if models that never saw each other's work land on the same
   conclusion, that is real signal, not one model just following another's lead.

2. **Anonymized peer review (optional, for genuinely contested questions).** Every answer
   gets shuffled and relabeled before any reviewer sees it, including its own author. Each
   reviewing model scores every answer, blind to which model produced which. This step is
   what catches a plausible-sounding but wrong answer: a review pass has caught fabricated
   facts, blocking logical errors, and factual slips that the models' own self-review never
   would have surfaced, precisely because the reviewer has no idea whose answer it is
   grading, including possibly its own.

3. **Chairman synthesis.** One model (or a human) sees all the anonymized answers, and
   where relevant, the peer reviews, and actively selects and weights, rather than blending
   everything together. A synthesis step that can say "this answer is right and that one is
   wrong" behaves very differently from an averaging step that can only ever produce
   something in between. Excluding a bad answer entirely is possible here; blending it in
   at a lower weight is not.

Skip step 2 when the question doesn't need it (routine work, or something checkable where
direct verification does the same job for less).

## Two findings on composition, and how far to trust them

**Composition probably matters more than headcount, but treat that as a working heuristic,
not a measured law.** In one run, adding a weaker voice "for diversity" appeared to pull a
strong model's result down rather than up. In a later run on a different question, the same
pairing showed no such effect: a strong model with cheaper models alongside it reached the
same answer as the strong model alone. Both were single instances with coarse scoring, and
the first sat inside the noise of its own judging. What survives both is narrower: prefer a
consistent tier over mixing tiers for the sake of variety, and do not assume an extra weak
voice is free.

**A cheap, well-matched council can approach an expensive solo model's quality.** Several
lower-cost models, run in the same parallel-then-synthesize pattern, can land close to what
one high-end model produces alone. This is the more useful, durable finding of the two: it
says less about which specific model is "best" right now (that changes every few months)
and more about a structural property, tier consistency beats raw model prestige, that keeps
holding as new models replace old ones.

## One operational lesson worth carrying over

Give every model an explicit instruction to work only from the brief it was given: no
looking things up, no exploring files or tools, unless its job specifically requires that.
Without this, a model will sometimes go looking for context on its own, find something it
was not meant to see (the grading rubric, another seat's answer, background it should be
blind to), and quietly contaminate its own answer. An explicit "work only from this prompt"
instruction, stated up front, prevents this reliably. Leaving it implicit does not.

## What this buys you, in practice

The value of a council usually isn't "smarter individual thinking" from any single model.
It's coverage and explicitness: catching a gap one model would have skipped, forcing a
verdict where a single answer might have stayed vague, and surfacing genuine disagreement
instead of quietly smoothing it into one confident-sounding voice. A single model gives you
one answer, stated with total confidence regardless of how contested the question actually
is. A council, done this way, tells you where the real disagreement lives, and lets you see
whether it resolved for a good reason or just because one voice was loudest.

## When a council doesn't pay off

On a tightly constrained question, it often will not change the answer.

Running one real decision four ways (a strong model alone; that strong model with three other
capable ones; the same strong model with cheaper models alongside it; and those same cheaper
models with a lighter model leading, no strong model in the room) produced the same
recommendation in all four cases. The differences that
did appear were second-order. When the constraints do most of the work, the decision is close
to forced, and extra heads have nothing to move.

Where the multi-head structure did visibly add something was earlier, in what got asked.
Given a chance to request outside information before answering, each council asked eight
questions; the single model asked five, and all five were already inside the councils' eight,
never the reverse. Breadth is not automatically better, one council's broader list still
missed the single most decision-relevant fact, but it is where the difference showed up.

So: reach for a council when the question is genuinely open and you expect real disagreement,
or when the research surface matters. On a decision the constraints already narrow, one good
model is likely enough. N=1 on both counts, with coarse scoring, so treat this as a reason to
check rather than a finding to lean on.

## Related work — this is a synthesis and a stress-test, not a new invention

Two lines of research motivate this directly:

- **Mixture-of-Agents** (Wang et al., 2024, [arXiv:2406.04692](https://arxiv.org/abs/2406.04692))
  — showed that layering several LLMs, each building on the previous layer's outputs, can
  outperform a single frontier model on standard benchmarks.
- **Multiagent debate** (Du et al., ICML 2024, [arXiv:2305.14325](https://arxiv.org/abs/2305.14325))
  — showed that having several model instances debate and critique each other's reasoning
  over multiple rounds measurably reduces hallucination and improves factual accuracy,
  compared to a single model working alone.

It isn't only an academic idea, either. **Perplexity shipped a feature literally called
"Model Council" in 2026** — send one question to several models at once, see where they
agree or diverge. Worth naming plainly rather than glossing over: the naming overlap with
this repo is coincidental, not a copy in either direction. **Microsoft 365 Copilot** does
something related but distinct: it orchestrates across multiple model providers (OpenAI,
Anthropic, Google), mostly as *routing* — picking which single model answers a given
request — rather than running several in parallel and actively synthesizing a combined
verdict. Routing answers "which one model should handle this"; a council answers "what do
several independently-reasoning models converge on, and where do they genuinely split."
Different questions, both legitimate, easy to conflate.

What this repo adds on top of that: a specific, opinionated implementation (parallel
dispatch → optional anonymized peer review → active chairman synthesis, not averaging),
plus a handful of operational lessons that only surface from actually running it
repeatedly — where mixing model tiers helps or hurts, why explicit isolation instructions
matter, and why unanimous agreement isn't automatically safe to wave through. This is a
customization and stress-test of an existing idea for real, repeated use, not a claim to
have invented multi-model deliberation.
