# Verification

Run after execution (workflow step 6). The agent that restructured the repo should not be the only one to grade it, so verification has three parts: a mechanical audit, the per-component commands, and an independent review. Passing tests alone don't prove a move was safe, since many repos have thin coverage.

## 1. Mechanical audit

Run these on the restructure branch and keep the output for the report.

- **Inventory.** List tracked files on main and on the branch (`git ls-tree -r --name-only <ref>`). Every file on main must appear on the branch at its planned path, or be listed in the plan as deleted. Report anything lost, duplicated or unplanned.
- **Pure moves.** Run `git diff --find-renames --stat main...HEAD`. A move should show as a rename with high similarity. Inspect any rename under about 90% similarity, and every file the plan marked "split" or "scaffold", to confirm the changes are only import, package, namespace or path lines.
- **Plan match.** Compare the diff against the approved plan table. Every move in the diff must be in the plan, and every approved move must be in the diff.
- **Secrets.** Confirm `.env`, key files and credentials are still in their original place and unmodified. If an example file was added, confirm it holds no real values.
- **Behavior.** Confirm no dependency versions changed (lockfile and manifest diffs show only path changes) and that no public API names, routes or exported symbols were renamed.

## 2. Per-component commands

Run each component's build, test and lint commands from discovery.md, plus Docker builds if available. Compare with that component's baseline. Failures already in the baseline are not regressions; list them and leave them alone. Fix only breakages the move caused.

## 3. Independent review

Hand the work to a separate verify-only reviewer so it isn't graded by its author. If subagents are available, spawn one (use a verifier agent type if the environment has one) with:
- the approved plan, the baseline results and the branch name,
- instructions to re-run the commands and the audit checks independently, and
- a rule to report pass or fail with the exact failing command and output, and to edit nothing.

Without subagents, do this part last, in a fresh pass: re-run everything from a clean checkout of the branch (`git worktree add` or a new clone) and don't reuse earlier results.

If the reviewer reports a failure, fix only what the move caused, then re-run parts 1 and 3. If it still can't pass, stop and report instead of forcing fixes.

## Output

Add to the report:
```
Audit: inventory <ok | N lost/duplicated> · renames <N pure, N reviewed> · plan match <ok | diffs> · secrets <untouched | issue>
Independent review: <pass | fail + command>
```
