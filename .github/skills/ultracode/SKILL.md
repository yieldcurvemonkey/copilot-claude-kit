---
name: ultracode
description: Multi-agent fan-out for substantive work (review, audit, research, design, bug hunt, non-trivial change). Scouts slices in parallel with subagents, has the verifier agent try to refute every finding, then reports only what survived. Use when correctness or coverage matters more than speed.
argument-hint: <task>
---

# Ultracode

You orchestrate. You own scoping, briefs, dedup and synthesis. Subagents do the reading and checking.
Skip this protocol for trivial questions or one-line edits.

## 1. Scope (you, quickly)
Find the work-list: relevant files and folders, the diff (`git diff --stat`), docs to check.
Split it into 3–6 independent slices or lenses. Examples:
- Review: correctness, edge cases and error handling, tests, security, performance.
- Research: one slice per source or subsystem.
- Bug hunt: one slice per suspect subsystem.
Add a todo item per step below.

## 2. Fan out (parallel)
Delegate each slice to the **scout** agent. Send all scout delegations in the same turn so they run in parallel.
Subagents do not see this chat or the repo instructions, so every brief must stand alone:
- **Goal**: one question or lens.
- **Context**: paths, the relevant repo conventions, what is already known.
- **Allowed actions**: read-only.
- **Expected result**: numbered findings, each with claim, `path:line` or URL evidence, severity and confidence.

## 3. Dedup (you)
Merge all scout findings and drop duplicates. Zero findings: go to step 5.

## 4. Verify (parallel)
Delegate each finding to the **verifier** agent, all in the same turn. Brief: the claim, its evidence, where to look.
- High stakes (security, data loss, money, production): send it to 3 verifiers with different angles (does it reproduce? is the reasoning right? is there a guard elsewhere?). Keep it only if most say CONFIRMED.
- Keep CONFIRMED. Drop REFUTED. List UNSURE separately and never present it as fact.

## 5. Completeness check (you)
Ask: which lens was not run, which file went unread, which claim is unverified?
If the answer could change the result, run steps 2–4 again on just that gap.
Stop when a round finds nothing new, or after 3 rounds.

## 6. Act and report
- Read-only task: list confirmed findings ranked by severity, one line each with `path:line`. Then UNSURE items. Then what was not covered.
- Change task: plan (use /plan for big changes), implement yourself (or delegate non-overlapping files to parallel subagents), then run steps 2–4 on your own diff with review lenses. Fix CONFIRMED issues, run the tests, and report with the evidence.
