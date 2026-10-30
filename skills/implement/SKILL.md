---
name: implement
description: Use when invoked directly, or for any non-trivial code change. Refines requirements, explores the code, then runs an implement → review loop with subagents via a handover folder.
argument-hint: <what to implement, optional if clear from context>
---

# implement

Create a handover folder with `mktemp -d -t aitools.implement.XXXXXX` and note the folder path.

All subagents see nothing but the handover folder and the workdir, so every handover must be self-contained.

## 1. Refine the requirements

- From this session collect:
  - the **goal**: what the user wants in the end
  - the **decisions** made along the way: chosen approaches, rejected alternatives, constraints, non-goals
- Then look for gaps:
  - acceptance criteria
  - scope
  - choices
  - contradictions
- Summarize your findings.
- Ask the user about unclear points and wait for the answers before continuing.
- Store the summary of the requirements as `requirement.md` in the handover folder and link that file to the user.

## 2. Explore subagents

Launch explore agents. They are read-only and may use only file read/search tools (no shell/CLI). Each gets the path of `requirement.md` and reports file paths with line numbers. Save each result in the handover folder.

First:

- **dataflow** → `explore.dataflow.md`: find the entry points, call paths, data flow and control flow relevant to the requirements: where the affected data is created, transformed and consumed, and which branches/conditions govern it.

Then, with `explore.dataflow.md` as additional input:

- **modules** → `explore.modules.md`: find the modules, classes, dependencies and class hierarchies that the requirements affect, and how they depend on each other.

## 3. Build the handover

Write `implement.md` into the handover folder, based on `requirement.md` and the explore results:

- **Goal**: what must be true when done
- **Plan / architecture notes**: approach, decisions already made, constraints, non-goals
- **Important files**: paths and why they matter
- **Tests to run when done**: exact commands and expected results
- **Project rules**: e.g. from AGENTS.md / CLAUDE.md

Reference `requirement.md`, `explore.dataflow.md` and `explore.modules.md` from `implement.md`. Put other long material in extra files in the same folder and reference it too.

## 4. Implement / review loop

Rounds are numbered `<n>` = 0, 1, 2, … Each round's subagents get the paths of all files in the handover folder. Only their main file is must-read; the others are for reference as needed.

### 4a. Implement

Round 0: launch an implementation subagent. Its main file is `implement.md`.

> 1. **Read the handover.** Read `implement.md` and every file it references, transitively.
> 2. **Implement.** Read the important files first. Follow the plan, but deviate if it's wrong or a better way shows up. Note each deviation and why. Also note any pitfalls, bugs or constraints you run into.
> 3. **Verify.** Run the tests from the handover and fix the failures your change caused.
> 4. **Report.** End with a final message (also written to `implement.0.result.md` in the handover folder):
>    - **Status**: done / partial / blocked
>    - **Result**: what changed and which files
>    - **Verification**: each test: passed / failed / not run, and why
>    - **Deviations**: plan → what was done → why
>    - **Pitfalls, bugs, constraints, additional notes**

Later rounds: continue the session of the round-0 implementation subagent (don't start a new one). Its main file is `implement.<n-1>.review.md`.

> 1. **Read** `implement.<n-1>.review.md`.
> 2. **Fix** each finding. If you think a finding is wrong or not worth fixing, skip it and say why.
> 3. **Verify.** Run the tests from the handover again.
> 4. **Report.** End with a final message (also written to `implement.<n>.result.md` in the handover folder): for each finding, fixed / skipped and why, plus files changed, test results, and new pitfalls or notes.

### 4b. Review

Launch a review subagent of the explore type: read-only, file read/search tools only. Save its final message as `implement.<n>.review.md`. Its main file is `implement.<n>.result.md`.

> 1. **Read** the main file, then the changed files it lists. Consult the other handover files as needed (requirements, plan, earlier reviews and results).
> 2. **Review** the changes against the requirements: correctness, missed requirements, edge cases, unjustified deviations, consistency with the surrounding code, missing tests. For findings skipped earlier: if the reason is obvious, accept it and don't raise them again; if you disagree, raise them again with severity increased by one level (minor → major → blocker).
> 3. **Report** a list of findings, most severe first. Each one has: severity (blocker / major / minor), file:line, the problem, and a suggested fix. If there are no findings, say so.

### 4c. Decide

- **Done** if the review has no findings at all, or if this review and the previous one both have no blocker/major findings.
- **Ask the user** for guidance every 3 rounds (before starting round 3, 6, …): show the open findings and what was tried so far.
- Otherwise start the next round at 4a.

## 5. Relay the result

Give the user a short summary: status, what changed, deviations, test results, number of rounds, open findings, and notable pitfalls. Link the handover folder for the full reports.
