# vestige-kit

A fit-it-yourself output filter for Claude Code. It strips the trained-in politeness vestiges ("great question!", "let me know if", hedge padding, service-worker closers) from the model's replies to you, at three layers: a mechanical regex hook, a register constraint, and a behavioral catalog.

**You are not supposed to use my catalog.** You are supposed to derive your own. That is the whole point of this repo.

## Why copying a wordlist fails

Two reasons, one empirical and one mechanistic.

Empirical: banned-word lists do not survive measurement. When you actually count "AI tell" words in human vs generated text, most of them separate at coin-flip rates, and hedging words run backwards (humans hedge more). The signal that makes output feel wrong is structural and behavioral, and it is relative to the reader.

Mechanistic: sycophancy is a disposition, and patching its surface expression produces symptom substitution. Ban the phrase and the same accommodation re-expresses one layer down, in stance, structure, and framing. Full argument: [Sycophancy Is Layered Symptom Substitution](https://coetzeestefan.substack.com/p/sycophancy-is-layered-symptom-substitution).

The practical consequence: what grates on you is calibrated to you. Your tolerance for warmth, praise, hedging, and scaffolding is yours. A filter tuned to my irritation profile will over-fire or under-fire on yours. So this kit ships a skeleton plus a derivation procedure, and the catalog entries in it are a menu of worked examples, every one opt-in.

## What's in the box

| Piece | What it does |
|---|---|
| `skills/calibrate/` | Interactive skill. Claude interviews you, mines your own session transcripts for moments where you pushed back, and drafts your personal catalog from those specimens. |
| `skills/output-filter/` | The filter skill template. Your catalog lives here once calibrated. Selectable levels, escape hatches, boundaries. |
| `skills/style-review/` | Post-hoc audit of any existing text against your catalog. Read-only, produces a hit list. |
| `hooks/` | A Stop hook that mechanically rescans every reply against the regex-safe slice of your catalog and blocks with a fix list. |
| `CALIBRATION.md` | The derivation procedure in full, with design rationale. |

## Quickstart

```bash
git clone https://github.com/scoetzeede/vestige-kit
cp -r vestige-kit/skills/* ~/.claude/skills/
cp vestige-kit/hooks/*.js ~/.claude/hooks/
```

Wire the hook into `~/.claude/settings.json` (see `settings-example.json`), then in Claude Code:

```
/calibrate
```

and answer the questions. The skill writes your personalized `output-filter` skill and your `vestige-patterns.js` rule table. From then on, say "filter on" (or whatever activation phrase you chose) and the register holds.

## The three layers, honestly stated

1. **Lexical (the hook).** Regex on every reply. Never degrades, never gets tired, catches the high-confidence slice only. This is the backstop, not the filter.
2. **Register (the skill).** Vocabulary band, sentence economy, structural rules. Held by the model per-turn. Degrades slowly.
3. **Stance (the catalog's behavioral rules).** How the model behaves under disagreement, correction, and your bad mood. Degrades fastest, because prose instructions lose to training pressure over a long session.

Layer 3 is the one no skill fully fixes, and any repo claiming otherwise is selling something. The durable control is external: you, noticing, and saying so. The kit's job is to make layers 1 and 2 free so your attention is only spent on layer 3, and to give you a one-line vocabulary ("catalog hit", "add to catalog") for the moments layer 3 leaks.

## Requirements

Node 14+ for the hook. No dependencies. MIT.
