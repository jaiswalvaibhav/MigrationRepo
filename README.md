# Jira Slice Migration: Final Prompts for Each Phase

What this version assumes:

- Repo B is read-only everywhere. All documents are written into repo A's `docs/` folder.
- B is analyzed as a slice (Jira UI + analyst agent + the orchestration path that runs it), not as a whole repo.
- The final goal is explicit: the Jira workflow must work end to end in repo A.

Replace anything in `[brackets]`. Open A and B as a multi-root workspace. After every phase, run `git -C <repoB> status` and confirm it is clean.

---

## Ground rules block (paste at the top of Phases 2 to 7)

```
GROUND RULES (apply to everything below)

Repos: A = target repo (baseline design). B = source repo (READ-ONLY).

1. Never create, edit, move, or delete any file in repo B. Read it only.
   All documents and code output go into repo A.
2. Repo A's orchestration design and agent workflow is the baseline.
   B's code is COPIED and ADAPTED into A's design. Do not change A's
   existing orchestration behavior.
3. Scope is only the "Jira workflow slice" of B: the Jira UI, the analyst
   agent, and the orchestration path that runs them. Ignore the rest of B.
4. The success criterion: the Jira workflow (UI + orchestration + agents)
   works end to end inside repo A after migration.
5. If B does something clearly better than A, or something A lacks, do not
   silently adopt it. List it under "Proposed exceptions" with evidence.
   I approve each one.
6. Cite file path and line range for every claim. Write UNCLEAR or
   UNVERIFIED instead of guessing.
```

---

## Phase 1A: Inventory of repo A (Sonnet 5, high)

Open repo A. Read-only, except for creating the one doc.

```
Read-only task. Create only docs/arch-A.md in repo A. Change nothing else.

Document how repo A works so another codebase can be adapted into it.

Sections:
1. Purpose, folder structure (one line each), entry points
2. UI: framework, main screens/components, routing, how it calls the backend
3. Agents: for EACH agent give name, role, prompt location, tools, inputs, outputs
4. Orchestration: how a run starts, how control passes between agents,
   static vs LLM-decided routing, where shared state lives, how a run ends
5. Extension points: exactly how a new agent or a new workflow is added today
   (files to touch, interfaces to implement, registration steps)
6. Contracts: message formats, state shape, error format, config/env vars
7. Failure handling: retries, timeouts, partial failures, human-in-the-loop
8. Tests: what exists, how to run them
9. Weak spots or TODOs

Rules: cite file:line for every claim. Write UNCLEAR if unsure.
Open the files; do not infer from file names.
```

---

## Phase 1B: Inventory of the Jira slice in repo B (Sonnet 5, high)

Repo B must be completely untouched. The output goes into repo A.

```
Read-only task on repo B. Do NOT modify, create, or delete anything in
repo B. Write the output to docs/arch-B-jira-slice.md in repo A.

Scope: only the Jira workflow slice. Start from the Jira UI entry point and
follow the dependencies outward: what the UI calls, what the orchestrator
does for this workflow, which agents run, and what config/services they need.
Ignore everything in B not reachable from this workflow.

Sections:
1. Workflow description: step by step, what the user does in the Jira UI and
   what happens behind the scenes, in order. This is the behavior that
   must survive migration.
2. Slice manifest: every file in B that belongs to the slice, grouped as
   UI / orchestration / agents / shared utilities / config / tests.
   For each: path, one-line purpose, approximate size.
3. Analyst agent: role, prompt location, tools, inputs, outputs, model used
4. Orchestration for this workflow: triggers, control flow, state, routing,
   how results get back to the UI
5. Contracts: message formats, state shape, API endpoints the UI calls
6. External dependencies: Jira APIs/auth, LLM providers, packages, env vars
7. Failure handling in this workflow
8. Existing tests, and which are reusable
9. Coupling: places where the slice depends on B code outside the slice
   (list them; these need special handling)

Rules: cite file:line for every claim. Write UNCLEAR if unsure.
Do not summarize from file names alone.
```

---

## Phase 2a: Fit analysis (Opus, high or extra high)

```
[PASTE GROUND RULES]

Read-only task. Write docs/fit-analysis.md in repo A.

Inputs: docs/arch-A.md, docs/arch-B-jira-slice.md. Open the source files in
both repos whenever you need to verify a claim.

Part 1. Mapping table, one row per element of B's slice:
| B element | Closest thing in A | Fit (Maps directly / Needs adaptation /
Conflicts / No equivalent in A) | Adaptation needed | Risk |

Part 2. Contradictions: where B's orchestration or agent behavior would
differ from A's for the same input, and what the user would see. Say how
each is resolved in favor of A's design.

Part 3. Proposed exceptions: where B's approach seems better than A's.
For each: evidence, what we gain by adopting it, what we risk, and a
recommendation. Do not adopt anything yourself.

Part 4. Coupling and dependencies: for each place the slice depends on
code outside the slice (from section 9 of the B doc), say whether to
copy it, replace it with A's equivalent, or rewrite it.

Part 5. Gaps: anything A needs (new extension point, new config, new
dependency) so the workflow can run. Keep changes to A's core
orchestration to the minimum, and say exactly what they are.
```

---

## Phase 2b: Workflow scenarios (Sonnet 5, high)

```
[PASTE GROUND RULES]

Read-only task. Write docs/workflow-scenarios.md in repo A.

Using docs/arch-B-jira-slice.md and the B source, define 6 acceptance
scenarios for the Jira workflow: [your main happy path], 1 ambiguous or
incomplete ticket, 1 Jira API failure, 1 agent returning bad output,
1 long-running case, 1 user retry or cancel case.

For each scenario give:
- Starting state and user actions
- Expected result in B today: step by step, which component runs, what it
  receives and returns, what the UI shows (with file:line)
- Expected result in A after migration, based on A's design
- Differences between the two, and whether each is acceptable

These scenarios will be used as the end-to-end acceptance tests, so write
the expected behavior precisely and observably.
```

---

## Phase 3: Decision record (Opus, same chat as 2a, extra high)

```
[PASTE GROUND RULES]

Using docs/fit-analysis.md and docs/workflow-scenarios.md, write
docs/decision-record.md in repo A. Be decisive: no "either is fine".

Sections:
1. Baseline: a summary of A's design rules that all ported code must follow
2. Migration manifest: for every file in B's slice, one decision:
   COPY AS-IS / ADAPT / REWRITE / REPLACE WITH A's EQUIVALENT / SKIP,
   with target path in A and a reason
3. Adaptation map: for each ADAPT or REWRITE item, before and after
   (B's interface to A's interface), with file refs
4. Shared contracts: the agent interface, message format, state shape,
   and error format, as real code types or JSON schemas
5. UI placement: where the Jira UI lives in A's UI, routing, navigation,
   and how it calls A's backend
6. Config and dependencies: new env vars, packages, Jira credentials
   handling (never copy secrets; list names only)
7. Changes to A's existing code: the complete list, each justified.
   Anything not listed here must not change.
8. Approved exceptions: leave this section empty; I will fill it in
9. Acceptance criteria: reference the scenarios in docs/workflow-scenarios.md
10. Open questions I must answer before implementation (max 5)
11. Out of scope

Every decision must link to its supporting evidence.
```

Review this yourself, fill in section 8, and answer section 10 before continuing.

---

## Phase 4: Merge plan (Sonnet 5, extra high)

```
[PASTE GROUND RULES]

Read-only task. Write docs/merge-plan.md in repo A from
docs/decision-record.md.

Break the work into ordered steps, each small enough for one reviewable PR.
For each step give:
- Goal (one sentence)
- Files to create/change in repo A (nothing in B)
- Depends on which earlier steps
- Tests that must pass before and after
- Risk (low/med/high) and rollback
- Any change to A's existing orchestration (flag clearly; should be minimal)

Required order:
1. Characterization tests for A's current orchestration and agent behavior
   (so we can prove nothing in A changed)
2. Minimal extension points or contracts in A (only what the decision
   record lists)
3. Port the analyst agent, adapted to A's agent interface
4. Wire the Jira workflow into A's orchestration
5. Port the Jira UI into A's UI
6. End-to-end tests from docs/workflow-scenarios.md running against A
7. Cleanup of anything temporary in A (never B)

You may add steps but not reorder these.
```

---

## Phase 5: Implementation (Sonnet 5, medium or high, fresh chat per step)

```
[PASTE GROUND RULES]

Implement step [N] of docs/merge-plan.md only.

Read first: docs/decision-record.md, the step [N] section of
docs/merge-plan.md, and the files that step lists. When porting from B,
read the B files and write adapted copies into A.

Rules:
- Follow the contracts in the decision record exactly. If something is
  missing or conflicts, stop and ask me instead of choosing.
- Touch only the files listed for this step. No unrelated refactors.
- Write or update tests first, then the code.
- Run the tests and show me the results.
- Never include secrets or tokens from B; use env var names from the
  decision record.

When done, list: files changed, tests run with results, and any deviation
from the plan (should be none).
```

---

## Phase 6: Review of each step (GPT 5.4 or Opus 4.8, high)

```
[PASTE GROUND RULES]

Review the current git diff in repo A against docs/decision-record.md and
step [N] of docs/merge-plan.md. Do not edit any files.

Check:
1. Scope: any extra or missing changes versus the plan?
2. Contracts: followed exactly?
3. Did this change any existing behavior of A's orchestration or agents?
   Flag each as a blocker unless the decision record approves it.
4. Fidelity to B's workflow: is anything from B's behavior lost or changed
   that the scenarios depend on?
5. Error and retry paths: still handled?
6. Secrets, hardcoded URLs, or B-specific paths copied in by mistake?
7. Missing or weak tests

Output findings with severity (blocker / should-fix / nit) and file:line.
```

---

## Phase 7: Final acceptance (Sonnet 5, high, after the last step)

```
[PASTE GROUND RULES]

Final acceptance check. Do not change code in either repo; write
docs/acceptance-report.md in repo A.

1. For each scenario in docs/workflow-scenarios.md, run it against repo A
   (using the e2e tests, or by tracing and running the app if tests don't
   cover it). Report PASS / FAIL with evidence.
2. Compare the observed behavior in A with the expected behavior recorded
   from B. List every difference and say whether the decision record
   approved it.
3. Run A's characterization tests from step 1 and report whether any
   existing behavior changed.
4. Confirm that repo B has no modifications (run git status in B).
5. Final verdict: is the Jira workflow working in repo A? List the
   remaining gaps, if any.
```

---

## Practical notes

- Commit `docs/` in repo A after each phase. If a chat goes wrong, start a new one and point it at the files.
- Because the scope is a slice, 200k context should be enough everywhere. Skip the 1M option unless Phase 1B shows the slice is bigger than expected.
- Phase 1B is the most important phase. If the slice manifest is incomplete (a missing shared utility, for example), the problem will only show up at Phase 5. Skim it yourself and ask the model to re-check any section that looks thin.
