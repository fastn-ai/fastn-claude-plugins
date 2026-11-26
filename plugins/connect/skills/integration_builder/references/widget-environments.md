# Widget environments

> Read this only when the user asks to release an integration to an environment (Live, Staging, Production or any other environment of their workspace), to promote it from one to another, to roll it back, or to review a promotion. Building and testing never deploy: a build ends with the widget as the partner edits it, which is what customers in `test` see.

A deployment freezes the widget (its name, description, connectors, workflow and trigger refs, hooks, activation mode, settings form, How it works override), its config template and its translations, together with the published version of every workflow the widget reaches (workflow refs, hooks, trigger targets, config option sources and every sub-flow they call). Customers in that environment then see and run exactly that, whatever the partner edits afterwards.

## What customers see

| The customer's environment | The widget | They see |
|---|---|---|
| `test` | any | the widget as it is being edited |
| a deployed environment | deployed there | that deployment, translated from its own translations |
| a deployed environment | deployed elsewhere, not here | nothing: left out of the list, 404 by id, and creating a setup of it or copying its template is 404 `WIDGET_NOT_FOUND` |
| any | never deployed anywhere | the widget as it is being edited (nothing changes for existing customers until the first deploy) |

The embed's environment is its `env` attribute (default `test`), held to the token's ceiling. A setup made in a deployed environment records it (`envSlug`), is built from that deployment (activation mode, connector pins, the config copy from the frozen template) and runs its hooks and triggers there.

## Tools

- `fastnPlatform__listWidgetDeployments` { id }: every environment of the workspace with its active deployment, history and `status`: `deployed`, `edited` (the widget changed since), `not_deployed`; plus `pendingPromotions`.
- `fastnPlatform__deployWidget` { id, env, dryRun? }: deploys the widget as last saved, with the latest published version of each workflow. **Always call it with `dryRun: true` first** and show the user the plan (each workflow, the version going out and the one there now). Deploy only after the user confirms.
- `fastnPlatform__promoteWidget` { id, from, to, message? }: copies `from`'s active deployment to `to` with the same workflow versions. Into an environment that requires review it answers 202 with a pending promotion instead.
- `fastnPlatform__rollbackWidget` { id, env, deploymentId? }: reactivates an earlier deployment (default: the one before the active one) and redeploys its workflow versions.
- `fastnPlatform__listWidgetPromotions` { id }, `fastnPlatform__approveWidgetPromotion` / `fastnPlatform__rejectWidgetPromotion` { id, promotionId, note? }, `fastnPlatform__cancelWidgetPromotion` { id, promotionId }.

## Rules

- Before `deployWidget` or `promoteWidget` to Live where the widget already has a deployment, run `regression_qa` (load its SKILL.md). Deploy only on PASS or PASS WITH WARNINGS; otherwise show its QA REPORT.
- Deploying, promoting and rolling back change what live customers run. A first deployment or a rollback happens only when the user asks for that environment, after showing the dry run, and never as part of a build. An update to a widget already live is deployed only after `regression_qa` returns PASS or PASS WITH WARNINGS and the user says to deploy.
- A workflow never published stops the deploy with 409 `WORKFLOW_NOT_PUBLISHED` naming it: publish it (`fastnPlatform__publishWorkflow`) after its suite passes, then deploy again.
- An environment that requires review cannot be deployed to directly (409 `ENVIRONMENT_REQUIRES_REVIEW`): deploy to another environment and promote from it.
- The requester cannot approve their own promotion (403 `PROMOTION_SELF_REVIEW`): someone else reviews it. Never approve one the user did not ask you to review.
- 409 `DEPLOY_IN_PROGRESS`: another deploy of this widget to that environment is running. Wait and retry once.
- 403 `ENV_NOT_PERMITTED`: the caller's key is pinned to another environment. 403 `WIDGET_NOT_IN_ACCOUNT`: an inherited widget, which only its own workspace deploys (fork it first, `references/forks.md`).
- `WIDGET_DEPLOY_FAILED` lists `restored` and `notRestored`: report both. Workflows in `notRestored` need the user's attention.
- After any of these, read back with `fastnPlatform__listWidgetDeployments` and report the environment's status and version.
