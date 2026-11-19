# Council scorecard — schema

[香港廣東話](scorecard_schema_zhHK.md)

A council is only as trustworthy as the track record you keep on it. Without a running
scorecard, "which models are actually good at what" reverts to a guess re-made every time.
This file defines the schema; keep your own real rows in your own copy, not in a public
repo.

## Columns

| Column | Meaning |
|---|---|
| Date | When the council ran |
| Task type | Category of question (writing, code, finance-style judgment, methodology critique, etc.) |
| Member | Which seat this row is about |
| Avg peer rank | Average rank given by the OTHER anonymized reviewers (1 = best; blank if no peer-review stage ran) |
| Distinct contribution | One phrase: what did this seat catch or add that the others didn't |
| Verdict | Did the human running the council agree with how this seat's contribution held up |

## Why this shape, specifically

- **Peer rank, not self-rank.** A model's own opinion of its answer is worth little; what
  the *other* seats, blind to authorship, ranked it is the actual signal.
- **One phrase per row, not a full transcript.** The point is a scannable track record you
  can skim across dozens of runs, not a full archive of every answer (keep those
  separately, per-run, if you want them).
- **Verdict is a human call, always.** The scorecard records what happened; it doesn't
  decide on its own whether a seat is worth keeping. That stays a judgment call, made by
  whoever is actually accountable for the decisions the council informs.

## What to do with it over time

- **Retire what isn't earning its seat.** If a member consistently ranks last or adds
  nothing distinct across several runs, drop it, don't keep including it out of habit.
- **Promote what performs.** A model that repeatedly catches something the others miss
  earns a more central role (e.g., moving from an optional extra seat to a default one).
- **Re-test on model updates.** A version bump can change a model's real-world performance
  in either direction. Treat a new version as a new candidate, not an assumed continuation
  of the old one's track record.
- **Keep it per-project or centralize it, your choice** — a table that spans everything you
  use a council for gives the fullest picture; a table per project is easier to keep
  private if some of your work is more sensitive than the rest.
