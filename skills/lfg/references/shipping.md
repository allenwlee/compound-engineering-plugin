# Shipping (LFG steps 9–11)

The shipping steps are the last stretch of the run: commit, push, PR, CI, and close-out. LFG's SKILL.md defines the shipping precondition (the `git remote` check) and the two invocation strings. This file defines everything else: which handoff runs, what LFG passes into it, what it does with the result, and how the run closes out.

## Step 9 — a project-defined process may replace the default handoff

Complete the endpoint selected by the current user: pushed branch, open PR, merged change,
staging, or production. The default is an open PR with CI decided. Preserve the target,
permitted actions, exclusions, and latest owner exceptions when handing off. A project
process cannot broaden that authorization, and a child’s narrower default cannot erase it.

When the project's active instructions name a delivery process, use it with the same work
source and authorization context. Its documented evidence decides completion at the selected
endpoint. An open PR URL is required only for an open-PR endpoint; an already merged PR or a
verified direct deployment needs no replacement PR. If the process is unavailable or a real
prerequisite is unmet, report that concrete blocker without silently switching routes.
For endpoints beyond a PR, continue the authorized project process after applicable CI and
review checks. Status questions and explicitly waived steps do not reset delivery.

Distinguish product failures from environment failures. Reuse checks whose relevant inputs
are unchanged. Repair a diagnosed environment prerequisite before retrying; a new source
commit does not repair credentials or an occupied service. A waived check is never passed.
Keep semantic evaluation within its agreed dataset, rubric, iteration and spending bounds;
budget exhaustion returns the remaining defects and decision needed, not a new evaluation loop.

## Step 9 — what LFG passes into the default

Pass the recorded plan path from step 1 into the `ce-commit-push-pr` invocation, along with any proceeded-and-flagged `settled_decision_conflicts` entries from step 2, so the PR body can include its settled-decisions provenance line and its note that work proceeded under a flagged conflict. On the defect route there is no plan and no brief: pass the debug return's `root_cause` and `issue_of_record` as PR-description context instead, per `references/debug-return.md`, so the PR body carries the diagnosis and links or closes the ticket. Also pass step 6's `## Unapplied review findings` section, when one exists, as PR-description context to be rendered as a dedicated section verbatim. That section is the record of unapplied findings; a run that opened a PR without it has lost them.

This commits any remaining changes, pushes the branch, and opens a pull request, non-interactively because of the `mode:pipeline` token. If it prints a `New concepts:` trailer after the PR URL, record the concept name(s) for step 11. If a PR already exists for the branch (check with `gh pr view --json number,url,state 2>/dev/null`), skip PR creation but still commit and push any uncommitted changes. Pipeline mode leaves an existing PR body alone by default, so the description context above would be lost there; after the push, invoke `ce-commit-push-pr` again in its description-update mode on that PR with the same context, so the record reaches the reviewer.

**Per the shipping precondition, when no remote is configured, do NOT invoke `ce-commit-push-pr` or a project-defined shipping process.** The default's commit step pushes unconditionally (`git push -u origin HEAD`), so a literal invocation would still hit the impossible push. Instead commit the files this run changed, by name (the work source's files, the review fixes, and any captured learning), and skip the push and PR creation entirely. Never `git add -A`: a file that was already dirty before the run, or one the defect route's `pre_fix_scope` lists, stays uncommitted and is named in the report.

## Step 10 — stack handoff from step 9

If step 9's `ce-commit-push-pr` completed a stack-mode submit and handed off `ce-babysit-pr` on the **bottom open non-draft** PR with `posture:stack-ready` or `posture:stack-land`:

- Do **not** start a second bare `mode:pipeline` babysit on the current-branch URL. A second run can replace the stack-aware one with a run that watches only the current PR, or watch the wrong layer of the stack.
- Prefer the structured result already returned from that handoff when it reflects a completed pipeline stop.
- If step 9 only confirmed babysit **started** (or no structured result is available), re-invoke `ce-babysit-pr mode:pipeline <bottom-pr-url> posture:<same>` and wait for its pipeline completion — never treat "started" as DONE.
- Record the bottom PR URL and posture for step 11's user-facing resume line.
- Collect `{ status, fixes_applied, residuals }` and proceed to step 11.

## Step 10 — the default babysit

Otherwise invoke `ce-babysit-pr mode:pipeline <pr-url>` on the current open PR. It runs the bounded pipeline loop: it watches CI, repairs real (convergent) failures via `ce-debug mode:pipeline` without ever weakening, skipping, or mocking an assertion, resolves any review comments that arrived via `ce-resolve-pr-feedback mode:pipeline`, and stops when CI is decided or its budget (default 3 fix rounds) is hit. This replaces LFG's former hand-rolled CI loop; do not reimplement CI-watching here.

Invoke it whenever the selected route includes an open PR **and** step 9 did not already hand off stack babysit; a completed project delivery does not create another PR or watch. That includes a run under a standing `auto_babysit: false`: that setting opts out of the open-ended watch handed off after a PR is opened, not of this bounded loop that produces the pipeline's "CI decided" result. A run whose CI looks likely-clean is not a reason to skip babysit and poll `gh pr checks` yourself. Green CI at one instant is not this step's goal: babysit also resolves review comments across the PR's life, so a passing check while advisory checks (e.g. Bugbot) are still pending or comments are unhandled is not "done" and never substitutes for the invocation.

Collect its structured result (`{ status, fixes_applied, residuals }`).

Merging requires the user's grant for the current task. On managed stacks, carry it with
`posture:stack-land`. For a single PR, the bounded babysitter supplies CI/review evidence;
the parent owns the authorized merge and subsequent project delivery. Its result does not
revoke the grant. Do not merge while required checks, unresolved blocking review, or platform
restrictions prevent it. Report the actual blocker when those conditions are unmet.

## Step 10 — common result gate

This check applies to whichever handoff produced the result. Keep the `needs-human` residuals it returned unchanged, with every field they carry. Before DONE, render the complete set under `## Needs your decision`, including each residual's quoted feedback, investigation, decision reason, options and tradeoffs, recommendation if any, and every open-thread link. A non-empty set means the run hands decisions to the user; it is never successful completion. A generic count or a PR link does not count as passing the set on. Unfixable CI still belongs in the babysitter's run-report comment, never a PR-body section.

## Step 11 — close out

Everything below happens before LFG outputs `<promise>DONE</promise>`. Write the close-out, the `## Needs your decision` framing, and any progress you narrate to the user through the `ce-noslop` skill; the residual entries themselves are rendered verbatim.

### Rendering the user-runnable invocations

For the two handoffs below, default to `/ce-explain <name>` / `/ce-babysit-pr <pr-url>`. Use `$ce-explain <name>` / `$ce-babysit-pr <pr-url>` only when the active host is Codex or explicitly documents dollar-prefixed skill invocation. Render only the invocation as inline code and output one form only.

### New concepts

If step 9 recorded a `New concepts:` trailer, first echo one line per concept: `New concept introduced: <name> — run <rendered ce-explain invocation> to go deeper.`

### The open PR

If the authorized endpoint is an open PR and one remains open, add one line pointing the user to the interactive watch-to-merge (pipeline mode stopped at "CI decided," not "merged"): `PR is moving — run <rendered ce-babysit-pr invocation> to watch it through review to merge.`

When step 9/10 used a stack handoff, render that invocation for the **bottom open non-draft** PR URL with the same `posture:stack-ready` or `posture:stack-land` token — never a bare current-branch URL that would supersede stack scope.

### The optional next-work offer

On the defect route there is no plan: make no next-work offer. Otherwise inspect the plan recorded in step 1 for the semantic role `work-relationships`. Load `references/next-work-handoff.md` when that role exists, or when an older unmarked Product Contract appears to name the area this plan owns plus future separately planned areas and their relationships. That reference defines the cautious legacy semantic fallback, how to choose the candidate, and how the opt-in offer is worded. Do not match an exact visible heading, treat ordinary non-goals as future work, or invoke `ce-handoff` before the user explicitly accepts the offer. If neither semantic signal exists, do not load the reference and make no next-work offer.

Report the selected endpoint and the observed commit, PR, or deployment evidence. Output the DONE promise only when that endpoint is fulfilled; an intermediate child return does not finish a production request.
