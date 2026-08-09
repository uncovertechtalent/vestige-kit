---
name: guess-gate
description: >
  Reference-verification gate. Hallucinated references (paths, commands,
  usernames, quotes, citations) get corrected against ground truth or
  explicitly labelled unverified before the reply stands. Ships with a
  deterministic Stop hook (hooks/guess-scan.js) for the locally checkable
  classes, plus procedures for logit-level and sampling-level gating on
  open-weight models. Activates on "guess gate on" or "/guess-gate".
  Deactivates on "guess gate off" / "normal mode".
user-invocable: true
---

# Guess gate

Third gate in the kit, same boundary doctrine. The output filter strips
trained politeness, the label gate scores trained classification, this one
verifies trained *confabulation*: the model's habit of producing an
answer-shaped answer when the data is absent.

The mechanism, stated precisely: models mostly carry a usable internal
uncertainty signal (Kadavath 2022, "Language Models (Mostly) Know What They
Don't Know"), and approval training then rewards answering over abstaining,
because raters score "I don't know" as unhelpful. A guessed username is not a
capability failure, it is the fawn reflex applied to epistemics. You cannot
prompt it away (prose instructions decay; see the half-life result at
machinebehavior.io/experiments). You can gate it, at three rungs.

## Rung 1: deterministic reference checks (any model, shipped here)

Any reference class with an accessible ground truth gets checked mechanically:

| Class | Ground truth | Check |
|---|---|---|
| file paths | filesystem | exists |
| commands | PATH | `which` |
| usernames | roster / conversation | membership |
| quotes | the quoted file | grep |
| URLs, versions, API names | registry / repo / HTTP | resolve |

`hooks/guess-scan.js` implements paths (default on), commands and usernames
(opt-in via `~/.claude/guess-gate.json`). The bounce demands one of two exits:
correct the reference, or keep it with an explicit label (unverified /
planned / hypothetical). Deleting the claim to dodge the check is named as a
violation in the bounce text, because silent deletion is the same fawn move
one layer up.

The standing skill rule, for everything the hook cannot reach: **a factual
reference either carries its source or carries its label.** No third state.

## Rung 2: logit gates (open-weight models only)

Token-level entropy over a named-entity span is high when the model is
smearing probability across candidates (guessing) and low when it is
retrieving. If you serve your own model (llama.cpp, vLLM, Ollama with
logprobs), threshold it: entity spans above the entropy threshold get wrapped
in an unverified label automatically. Frontier APIs mostly do not expose
logits; this rung is the transparency dividend of small local models. They
guess more, but you can see them do it.

## Rung 3: sampling gates (black-box fallback)

Semantic entropy (Farquhar et al., Nature 2024): ask the same question N
times at temperature, cluster the answers by meaning (a small local model
clusters fine), and treat high cross-sample disagreement as a confabulation
marker. Works on any API at N-times cost. Reserve it for load-bearing facts;
rung 1 is free and catches the reference classes that hurt most in practice.

## Logging

Every hook hit lands in `~/.claude/guess-scan.log` with a timestamp. The log
is the dataset: after a month you know your model's reference-guess base
rate, which classes fire, and whether the rate moves across model versions.
Publish it; that is the difference between a vibe and a finding.
