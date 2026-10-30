---
name: implement
description: Hand an implementation task to a subagent. Refines the goal and decisions from the session, explores the affected code, builds a handover (goal, plan/architecture notes, important files, tests to run), launches a subagent to implement it, has the result reviewed and sends the findings back to the implementer, and relays the reports.
argument-hint: <what to implement, optional if clear from context>
---

# implement

Create a handover folder with `mktemp -d -t aitools.implement.XXXXXX` and note the folderpath. 

## 1. Refine the requirements

- From this session collect: 
  - the **goal**: what the user wants in the end
  - the **decisions** made along the way: chosen approaches, rejected alternatives, constraints, non-goals
- Then look for gaps: 
  - acceptance criteria, 
  - scope,
  - choices
  - contradictions
- Summarize your findings
- Ask the user about unclear points
- store the summary of the requirements to requirement.md in handover folder and link that file to the user

## 2. Explore subagents

Launch explore agents. They are read-only and may use only file read/search tools (no shell/CLI). Each gets the path of `requirement.md` and reports file paths with line numbers. Save each result in the handover folder.

First:

- **dataflow** → `explore.dataflow.md`: find the entry points, call paths, data flow and control flow relevant to the requirements: where the affected data is created, transformed and consumed, and which branches/conditions govern it.

Then, with `explore.dataflow.md` as additional input:

- **modules** → `explore.modules.md`: find the modules, classes, dependencies and class hierarchies that the requirements affect, and how they depend on each other.


## 3. Build the handover

Write `implement.md` into handover folder, based on `requirement.md` and the explore results: 

- **Goal**: what must be true when done
- **Plan / architecture notes**: approach, decisions already made, constraints, non-goals
- **Important files**: paths and why they matter
- **Tests to run when done**: exact commands and expected results
- **Project rules**: e.g. from AGENTS.md / CLAUDE.md

Reference `requirement.md`, `explore.dataflow.md` and `explore.modules.md` from `implement.md`. Put other long material in extra files in the same folder and reference it too.

The subagent sees nothing but the handover folder with implement.md and the workdir. 

## 4. implemtation subagent

Give it the instructions below, with `<implement>` set to the path of `implement.md`:

> 1. **Read the handover.** Read `<implement>` and every file it references, transitively.
> 2. **Implement.** Read the important files first. Follow the plan, but deviate if it's wrong or a better way shows up. Note each deviation and why. Also note any pitfalls, bugs or constraints you run into.
> 3. **Verify.** Run the tests from the handover and fix the failures your change caused.
> 4. **Report.** End with a final message (also written to `implement-result.md` in the handover folder):
>    - **Status**: done / partial / blocked
>    - **Result**: what changed and which files
>    - **Verification**: each test: passed / failed / not run, and why
>    - **Deviations**: plan → what was done → why
>    - **Pitfalls, bugs, constraints, additional notes**

## 5. Review subagent

Launch a read-only review subagent (it must not modify any files). Give it the instructions below, and save its final message as `review.<n>.md` in the handover folder (`<n>` = review round, starting at 1):

> 1. **Read** `<implement>`, every file it references, `implement-result.md` and any earlier `review.*.md` / `fix-result.*.md`. Then read the changed files.
> 2. **Review** the changes against the requirements: correctness, missed requirements, edge cases, unjustified deviations, consistency with the surrounding code, missing tests.
> 3. **Report** a list of findings, most severe first. Each one has: severity (blocker / major / minor), file:line, the problem, and a suggested fix. If there are no findings, say so.

## 6. Fix findings

If `review.<n>.md` has findings, send them back to the implementation subagent from step 4 (continue its session, don't start a new one) with these instructions:

> 1. **Read** `review.<n>.md`.
> 2. **Fix** each finding. If you think a finding is wrong or not worth fixing, skip it and say why.
> 3. **Verify.** Run the tests from the handover again.
> 4. **Report.** End with a final message (also written to `result.<n>.result.md` in the handover folder): for each finding, fixed / skipped and why, plus files changed, test results, and new pitfalls or notes.

Then go back to step 5 for the next review round. Repeat until a review has no findings. Before the 3rd fix round, stop and ask the user for guidance: show the open findings and what was tried so far.

## 7. Relay the result

Pass on to the user the implementation report, the review findings and fix reports of each round, and any open findings, along with the handover folder path.
