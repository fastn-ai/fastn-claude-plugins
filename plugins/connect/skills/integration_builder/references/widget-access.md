# Who sees which integration, and what they may do

> Read this only when the user wants to decide, per customer or per user, which integrations their hub shows and how (hidden, paused with a message, offered as an upgrade, some buttons hidden), or which users may connect, disconnect or configure. Without it every customer sees every enabled integration.

Both are workflows in the partner's own workspace, named in the org's hub settings. fastn runs them for a customer's session, caches the answer, and enforces it on the server as well as in both hubs.

## The widget filter (`widgetFilterWorkflowId`)

Input: `{ endOrgId, user { id, email, metadata }, language, partnerToken, widgets [{ id, slug, name }] }`.

Output: `{ expiresIn, widgets: [{ id, hidden, disabled, disabledMessage, subscribed, subscriptionUrl, hideActions }] }`. A widget left out of the answer is shown as usual.

| Field | Effect in the hub | Effect on the api |
|---|---|---|
| `hidden: true` | not listed | reads 404, any action 404 `WIDGET_NOT_FOUND` |
| `disabled: true` + `disabledMessage` | no Connect, the message instead (Manage kept once connected) | connect 403 `WIDGET_DISABLED` |
| `subscribed: false` + `subscriptionUrl` | no Connect, "Not included in your plan" with an Upgrade link (http(s) only) | connect 403 `WIDGET_NOT_SUBSCRIBED` |
| `hideActions: ["connect" \| "disconnect" \| "configure"]` | those buttons gone, with no message | that action 403 `ACTION_NOT_ALLOWED` |

Write `disabledMessage` in the reader's language from `input.language`.

## The role check (`accessWorkflowId`)

Input: `{ endOrgId, user { id, email, metadata }, language, partnerToken }`.

Output: `{ expiresIn, canConnect, canDisconnect, canConfigure, message }`. A flag that is not `true` refuses that action for this user: the button is gone, the hub shows `message` (on a card in place of Connect, on a connected integration's page under its setups), and the api answers 403 `ACTION_NOT_ALLOWED` with the message. Changing an existing setup's accounts counts as configure.

## Setting them

1. Build each as a workflow on the **instant** tier (BUILD ORDER in `references/build.md`, with test cases for every answer it can give), and publish it. Anything else is refused with 422 `HUB_SETTINGS_WORKFLOW_INVALID`.
2. `fastnPlatform__patchHubSettings` { patch: { widgetFilterWorkflowId: "wf_..." } } and/or { patch: { accessWorkflowId: "wf_..." } }. `null` turns one off. The builder's Features tab (Customer access) shows the same two settings.
3. Verify as a customer: open the hub with an embed token for a test customer and check each rule, then call the refused action and confirm the 403 or 404.

## Rules

- `partnerToken` is the host page's own user token (the embed's `partner-token` attribute). The workflow may call the partner's backend with it to look the user up. Never log it or return it.
- `expiresIn` is seconds, at most 3600. The answer is cached per customer, user, language and workflow, so a change to the workflow shows after that.
- If the workflow fails or answers in the wrong shape, nothing is shown: the filter makes the hub say it could not load the integrations (424 `WIDGET_FILTER_FAILED`), and the role check refuses every action with "We couldn't check your access." Test the failure cases before turning a hook on.
- A partner's own staff acting on a customer's setup from the dashboard is never refused. Only the customer's own session is checked.
