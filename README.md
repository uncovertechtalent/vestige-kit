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
| `skills/label-gate/` | Epistemic gate for diagnostic labels ("conspiracy theory", "debunked", "fringe"). The label becomes a trigger instead of a verdict: it stands only after the claim is scored on four form questions, printed with the answer. Same boundary-enforcement doctrine, different trained reflex. |
| `skills/guess-gate/` | Spot-checker for asserted references (paths, commands, usernames) in the final report, the message no downstream tool call verifies. Narrow by design: confabulation is a provenance problem and the general fix is retrieval-with-sources (vault-kit's job); this covers the locally checkable slice via `hooks/guess-scan.js`. Every factual reference either carries its source or carries its label. |
| `hooks/` | Stop hooks that mechanically rescan every reply: `vestige-scan.js` against the regex-safe slice of your catalog, `guess-scan.js` against checkable reference classes (path existence, PATH commands, username rosters). Plus `analyze-log.js` to turn the hit logs into retire/promote decisions. |
| `CALIBRATION.md` | The derivation procedure in full, with design rationale, the add-to-catalog maintenance loop, and a token-savings measurement. |
| `EXAMPLES.md` | Three example calibrations (engineering lead / technical writer / solo dev) showing the spectrum. Examples, not defaults. |

## Quickstart

```bash
git clone https://github.com/uncovertechtalent/vestige-kit
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

## Community and further reading

- Discussion, case studies, and replication attempts: [r/ModelBehavior](https://www.reddit.com/r/ModelBehavior/) — behavioral science for AI systems; bring receipts.
- The mechanism argument this kit implements: [Sycophancy Is Layered Symptom Substitution](https://coetzeestefan.substack.com/p/sycophancy-is-layered-symptom-substitution).
- Independent measurement that wordlists fail (the AUC numbers): [measured-humanizer](https://github.com/SadhvikChirunomula/measured-humanizer).
- Why the label gate exists, with the historical receipts: [the archive is a witness with interests](https://redd.it/1vjxclr) and [the harness cut](https://redd.it/1vjxn2r).
- The program's claims ledger and experiments: [machinebehavior.io](https://machinebehavior.io).

## Requirements

Node 14+ for the hook. No dependencies. MIT.
