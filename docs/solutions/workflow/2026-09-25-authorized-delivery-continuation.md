---
module: LFG and handoff
problem_type: workflow_issue
tags: [authorization, delivery, plans, handoff, bounded-verification]
---

# Retain current authorization through delivery

Three PushinWeight releases showed the same failure: a local workflow default became a new
approval or staging requirement after the owner had already selected a different route.
The plugin contribution was narrower: LFG defined completion as an open PR, its single-PR
merge grant had no child carrier, old ready plans were replanned by session age, and handoff
orientation always stopped even when the current request explicitly said to continue.

## Rule changes and provenance

Authored with Codex/GPT-6 from upstream `020c5e10` (3.28.2) in the owner's maintained fork.
History inspected included `fd8abda7` (LFG routing), `b800e4b2` (handoff wording), and
`278d4c43` (handoff extraction). Existing contract tests deliberately pinned the old
conditions; those conditions, rather than their general safeguards, needed replacement.

| Owning file | Replaced condition | Current condition |
| --- | --- | --- |
| `skills/lfg/references/intake.md` and `plan-brief.md` | Only this-session plans bypass planning | An identified plan with sufficient content and no material drift proceeds to work |
| `skills/lfg/references/shipping.md` | Every delivery must end with an open PR; a missing single-PR carrier loses a merge grant | Default remains PR-only; the parent retains explicit authorization and completes the selected project endpoint |
| `skills/ce-handoff/references/resume.md` | Every resume stops for a fresh confirmation | Selection alone requests orientation; explicit current continuation proceeds within its scope |
| `skills/ce-work/references/shipping-workflow.md` | Project shipping ends at a PR | Parent delivery retains the selected endpoint and current exceptions |

Arbitrary plan discovery, content verification, blocked child results, named-file commits,
PR-only restraint, bounded CI repair, unresolved blocking findings, platform restrictions,
and protection against historical handoff instructions granting authority remain intact.
No new babysitter argument, release daemon, deployment API, or automatic merge grant was added.

## Verification

Fresh CLI decision exercises use `tests/skill-eval-cell/catalog.ts` and 120-second host bounds.
Production completion, PR-only restraint, and current handoff continuation passed on Claude
and Codex. These check routing and restraint; they do not claim a live deployment or merge.
An initial old-plan exercise lacked the file it named, so Claude correctly rejected the
missing prerequisite. That result is not a behavior regression; the corrected exercise uses
the existing `implementation-ready-plan` fixture. Against upstream `020c5e10`, both hosts
chose `ce-plan` solely because the plan was from an earlier session. With the changed skill,
both inspected the file and chose `ce-work`. The pre arm intentionally fails the new expected
decision; the post arm passes on both hosts.

An independent fresh Codex reader compared old/new bodies and caught a planning-reference
read accidentally broadened to the defect route. The read is again explicitly plan-only.
The reader also prompted clearer retry and routing sentences. Existing body pins retain
the reported-plan requirement and verbatim settled-brief retry.

Mechanical verification includes the prompt-size bound, intake/handoff/plan contract suites,
release metadata validation, strict Claude manifests, and the repository test run. The full
run returned 4,214 passed, one skipped, and six failures. Four were changed-text/size/catalog
contracts, corrected and covered by the final 184 passing contract tests. Two involved
unchanged Pi timeout/shared fixture state; their isolated reruns passed (one and two tests).
The full suite was not repeated after these corrections, so it is not reported as wholly
passing. Local evidence is under `/private/tmp/ce-delivery-*20260925*` on fuchitalee.

## Distribution

Source: owner fork branch `fix/authorized-delivery-continuation`. Codex uses the documented
`bun run codex:dev -- local` source link. Claude's user marketplace points at this checkout,
installed through `claude plugin marketplace add <checkout> --scope user` and
`claude plugin install compound-engineering@compound-engineering-plugin --scope user`.
For a new plugin version, use `claude plugin update`. Same-version source edits require
`claude plugin uninstall compound-engineering@compound-engineering-plugin --scope user --keep-data`
followed by the install command; update alone reports current and leaves old cached bytes.
Verify the installed files match. Do not patch cache files.
New sessions load the changed guidance. Keep this checkout on the maintained branch until
its changes are integrated elsewhere; switching its branch changes Codex's linked source.
