# MigrationRepo
test Migration Repo to merge changes of 1 repo to another

Replace anything in [brackets] with your own details. Run Phases 1 to 3 in separate chats, and start a fresh chat for each step in Phase 5.

Phase 1: Inventory (run once per repo, Sonnet 5, high thinking)

Open repo A on its own, run this, then repeat in repo B with B and the second repo’s name.

Read-only task. Do not modify any source file. Create only docs/arch-A.md.

Goal: document how this repo works so it can be compared against another repo.

Cover these sections, in this order:
1. Purpose and top-level structure (folders, one line each, entry points)
2. UI: framework, main screens/components, how it calls the backend
3. Agents: for EACH agent give name, role, system prompt location, tools it can call, inputs, outputs
4. Orchestration: how agents are triggered, how control passes between them, whether routing is static or decided by an LLM, where shared state lives, how a run starts and ends
5. Failure handling: retries, timeouts, partial failures, human-in-the-loop points
6. Config, env vars, external services (Jira, LLM providers, DBs)
7. Tests: what exists, what is covered
8. Known weak spots or TODOs you notice in the code

Rules:
- Every claim must cite file path and line range, like src/orchestrator.py:40-95.
- If you are unsure, write "UNCLEAR" instead of guessing.
- Do not summarize from file names alone; open the files.
Phase 2a: Comparison matrix (Opus, high or extra high)

Open both repos as a multi-root workspace, or attach arch-A.md and arch-B.md.

Read-only task. Write docs/compare-orchestration.md.

Inputs: docs/arch-A.md, docs/arch-B.md. Open the actual source files whenever
you need to verify a claim.

Part 1. Overlap map: list every concept that exists in both repos (e.g. agent
coordination, state, routing, retries, prompts for the analyst). Mark each as
SAME, SIMILAR, or CONFLICTING.

Part 2. For each overlapping concept, fill one row:
| Concept | Repo A approach (with file refs) | Repo B approach (with file refs) | Winner (A/B/Neither/Merge) | Reason |

Judge only on these criteria: correctness, failure handling, testability,
extensibility, observability, and fit with the Jira analyst workflow.
Do not judge on code style.

Part 3. Contradictions: list places where the two repos would behave
differently for the same input, and what the user would see.

Part 4. Things only one repo has, and whether it should be kept.

Rules: cite file and line for every claim. Mark anything you could not verify
as UNVERIFIED. Where the winner is not obvious, say what evidence would settle it.
Phase 2b: Scenario tests (Sonnet 5, high thinking)
Read-only task. Write docs/scenario-traces.md.

Pick 5 realistic scenarios for [describe your use case, e.g. "analyst agent
reads a Jira ticket and proposes a plan"]. Include: 1 happy path, 1 ambiguous
input, 1 tool failure, 1 agent returning bad output, 1 long-running task.

For each scenario, trace step by step what repo A does and what repo B does:
which agent runs, what it receives, what it returns, what state changes.
Cite file and line for each step. Then state where the two diverge and which
behavior is better for the user, with a reason.

Do not run anything unless a test already exists for it. Trace from the code.
Phase 3: Decision record (Opus, same chat as 2a, extra high)
Using docs/compare-orchestration.md and docs/scenario-traces.md, write
docs/decision-record.md. Be decisive.

Sections:
1. Canonical orchestration: which repo's approach we keep, and why (3 to 5 bullets)
2. Ported in: specific pieces taken from the other repo (with file refs)
3. Deleted: what gets removed after the merge
4. Shared contracts: define the agent interface, message format, state shape,
   and error format. Give them as actual code types or JSON schemas.
5. Merge target: where the Jira UI lands inside repo A's UI, and how it talks
   to the backend
6. Open questions I must answer before implementation (max 5)
7. Out of scope

Rules: every decision must link to the evidence row that supports it.
Do not leave "either is fine" decisions open; pick one and say why.

Read this one yourself and answer section 6 before moving on.

Phase 4: Merge plan (Sonnet 5, extra high)
Read-only task. Write docs/merge-plan.md from docs/decision-record.md.

Break the merge into ordered steps. Each step must be small enough for one
reviewable PR. For each step give:
- Goal (one sentence)
- Files to create, change, or delete
- Dependencies on earlier steps
- Tests that must pass before and after
- Risk (low/med/high) and how to roll back
- A copy-pasteable prompt for the implementing agent

Required step order: (1) characterization tests for current behavior,
(2) shared contracts, (3) move orchestration, (4) port analyst agent,
(5) port Jira UI, (6) delete duplicates and dead code.
You may add steps but not reorder these.
Phase 5: Implementation (fresh chat per step, Sonnet 5, medium or high)

Use the same template for each step:

Implement step [N] of docs/merge-plan.md only.

Context files to read first: docs/decision-record.md, the step [N] section of
docs/merge-plan.md, and the files listed in that step.

Rules:
- Follow the contracts in the decision record exactly. If something conflicts
  or is missing, stop and ask me instead of choosing.
- Do not touch files outside this step's list.
- Do not refactor unrelated code.
- Write or update tests first, then the code.
- Run the tests and show me the results.

When finished, list: files changed, tests run and their results, and anything
you deviated from the plan on (should be none).
Phase 6: Review (different model, GPT 5.4 or Opus 4.8, high)
Review the current git diff against docs/decision-record.md and step [N] of
docs/merge-plan.md.

Check and report on each:
1. Does the diff match the planned scope? (extra or missing changes)
2. Does it follow the shared contracts exactly?
3. Any behavior from the old orchestration that was silently lost or changed?
4. Error and retry paths: still handled?
5. Missing or weak tests

Output a list of findings with severity (blocker / should-fix / nit) and the
file and line for each. Do not edit any files.
Two tips
Commit the docs/ folder after each phase. If a chat goes wrong, you start a new one and point it at the files instead of re-explaining.
If a phase output looks thin, don’t ask “are you sure?”. Ask it to re-check a specific section and cite file lines for each claim.
