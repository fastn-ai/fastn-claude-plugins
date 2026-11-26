# Release review: the user promotes to Live

> Read this at BUILD Step 9, after every flow's suite passed and was published, and whenever the user asks to take a workflow live. Promoting to Live is the user's decision: you hand them a review link, they press the button.

## Procedure

1. **Persist the results first.** The review page shows the flow's attached test cases with the results of its last validation run, so run the full suite and save it with `fastnPlatform__saveWorkflowValidation` (failures included). Publish the flow (`fastnPlatform__publishWorkflow`): the button promotes the newest published version. **Already live?** Then this is an update: run `regression_qa` first (UPDATES TO A LIVE INTEGRATION in SKILL.md) and create the review only on PASS or PASS WITH WARNINGS; otherwise show its QA REPORT.
2. **Create one review per flow** of the use case: `fastnPlatform__createReleaseReview { workflowId, message? }`. `message` is one plain line on what changed or what the flow does.
3. **Hand the user each `reviewUrl`.** Opening it shows the cases (passed, failed, untested) and a **Promote to Live** button that deploys the newest published version to the org's default (Live) environment after a confirmation. Tell them the button is theirs to press.
4. **If they say they promoted**, confirm with `fastnPlatform__getReleaseReview { id }` and report the outcome.

## Rules

- **Never take a workflow live yourself**, first build or update, even when you have the rights and the suite is green: the review link is the path you offer, unless the user asks you to deploy. An update also needs a `regression_qa` PASS first.
- **Read the verdict before handing over the link.** `ready` = hand it over. `failing`, `stale` (the code changed since the last run), `untested` or `no_tests` = fix and re-run the suite first; if the user still wants the link, tell them plainly what they would be promoting.
- **`canPromote: false`** means the link's creator lacks `workflows.deploy_prod`, so the button will not work. Say that someone with deploy rights must create and open the review.
- **The link opens without sign-in for 20 minutes; the review lasts 24 hours.** Expired link: call `fastnPlatform__createReleaseReview` again for a fresh one.
- **Status from `fastnPlatform__getReleaseReview`:** `pending` (not promoted yet), `promoted` (live), `pending_review` (Live requires a GitHub review: the promotion waits for that approval; tell the user).
- 404 = the workflow does not exist in this org (an inherited flow: fork it first, `references/forks.md`).
- Widget deployments to environments are separate: `references/widget-environments.md`.
