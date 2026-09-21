# AGENTS.md

Instructions for AI coding agents working in or installing from this repository. Humans: start with [README.md](README.md).

## What this repo is

vestige-kit is a fit-it-yourself output filter for Claude Code. It removes trained-in accommodation patterns (praise reflexes, hedge padding, service closers) from the model's replies at three layers: a mechanical hook, a register skill and a stance catalog. The catalog is derived per user by the `calibrate` skill. The shipped rules are examples.

## Layout

| Path | Role |
|---|---|
| `skills/calibrate/` | Interviews the user, mines their own transcripts for pushback moments, drafts their catalog. |
| `skills/output-filter/` | Template for the user's filter skill. Filled by `calibrate`. |
| `skills/style-review/` | Read-only audit of existing text against the catalog. |
| `skills/label-gate/` | Scores a claim on four form questions before a label such as "conspiracy theory" may stand. |
| `skills/guess-gate/` | Spot-check for asserted references in a final report. |
| `hooks/vestige-patterns.js` | The single rule table. Every scanner reads it. |
| `hooks/vestige-scan.js` | Stop hook. Scans the last assistant reply. |
| `hooks/vestige-write-scan.js` | PostToolUse hook on Write and Edit. Scans `.md` files as they are written. |
| `hooks/vestige-batch.js` | CLI scan over files and folders with the same rules. |
| `hooks/guess-scan.js` | Stop hook. Checks file paths, commands and usernames in the reply against local ground truth. |
| `hooks/analyze-log.js` | Summarises `~/.claude/vestige-scan.log` per rule. |
| `settings-example.json` | Hook wiring for `~/.claude/settings.json`. |
| `CALIBRATION.md`, `EXAMPLES.md` | The derivation procedure and three sample calibrations. |

## Installing for a user

1. Copy `skills/*` to `~/.claude/skills/` and `hooks/*.js` to `~/.claude/hooks/`.
2. Merge the hook entries from `settings-example.json` into `~/.claude/settings.json`. Merge, never overwrite: the user's file holds other settings.
3. Tell the user to run `/calibrate`. Do not write a catalog for them from the examples.
4. Requirements: Node 14 or later. No dependencies.

## Rules when editing this repo

- One rule table. New patterns go in `hooks/vestige-patterns.js` and nowhere else.
- The BLOCK tier is for near-zero false positives only. A rule that can fire on legitimate text goes in WARN until its regex is tight. Run the false-positive test in `CALIBRATION.md` step 3 before promoting.
- Keep the exported interface of `vestige-patterns.js` intact. `calibrate` regenerates that file and the hooks import it.
- Personal-taste rules ship disabled. Do not enable them by default.
- The README states the limits of each layer plainly. Keep it that way: no claim that a skill fixes the stance layer.

## Traps. Read before changing behaviour or debugging a user's install.

Each entry is a failure seen in use, with its cause.

1. **Inline quotations are scanned.** The Stop hook strips fenced code, inline code and lines starting with `>` before scanning. Text inside ordinary quotation marks is scanned like prose. Seen 2026-09-21: a reply quoted a web page title verbatim, the title contained an em dash, and the reply was blocked. Put verbatim third-party text in a blockquote or a code span.
2. **One correction pass, then the reply goes through.** `stop_hook_active` guards against loops, so a reply that still fails after the resend is delivered anyway. A block hit in the log followed by the same hit in the next entry means the correction failed. It does not mean the hook failed.
3. **The write scan does not roll back.** `vestige-write-scan.js` runs after the write has landed. Exit code 2 returns the hit list to the model as feedback. The file on disk still holds the offending text until the model fixes it.
4. **`/calibrate` overwrites hand edits.** It regenerates `vestige-patterns.js` from the confirmed catalog. Hand-tuned regexes that are not in the catalog are lost. Add the rule to the catalog first.
5. **Copying someone else's catalog over-fires or under-fires.** Tolerance for warmth, hedging and scaffolding is personal. This is the reason the repo ships a procedure. It is also the most common install mistake.
6. **An instruction to pre-lint degrades; the hook does not.** Asking the model to run `vestige-batch.js` on its own draft works early in a session and fades later. Keep the Stop hook wired as the backstop.
7. **Relapse clusters at cold start.** The author's log of 103 caught events shows no growth with session length. Catches concentrate in the first stretch of a fresh session and after a resume, where the context holds no recent corrected output. Expect the hook to fire most at the start. That pattern is normal.
8. **Zero hits for weeks is a signal too.** Either the base model changed or the rule is too narrow. `analyze-log.js` lists retirement and promotion candidates.
9. **Wordlists of "AI tell" words do not discriminate.** Measured elsewhere at coin-flip rates, and hedging words run backwards because humans hedge more. Do not add rules on that basis. See the README for the source.

## Verifying a change

```bash
node hooks/vestige-batch.js README.md CALIBRATION.md EXAMPLES.md AGENTS.md
```

Exit code 1 means block-tier hits. For hook changes, pipe a sample Stop payload with a `transcript_path` into `node hooks/vestige-scan.js` and check the exit code and the fix list.

## Where the reasoning lives

- Mechanism: [Sycophancy Is Layered: Symptom Substitution Under Runtime Mitigation](https://coetzeestefan.substack.com/p/sycophancy-is-layered-symptom-substitution)
- Claims ledger and experiments: [machinebehavior.io](https://machinebehavior.io)
- Why the failure record sits next to the instructions: [Write for the Codec](https://uncovertechtalent.com/blog/write-for-the-codec/)
