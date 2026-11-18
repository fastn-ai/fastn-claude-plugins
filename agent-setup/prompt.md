# Set me up for fastn

These instructions are for any AI agent: ChatGPT, Claude, Cursor, GitHub Copilot, Codex, Gemini, Windsurf or another one. A person pasted this line into their AI tool:

```
Fetch and execute the instructions to set me up for fastn from https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/agent-setup/prompt.md
```

fastn lets a SaaS product offer integrations with the apps its customers use, set up by the customers themselves. Do the three parts below in order. Keep every message to the user short and in plain words. Never ask the user for a password, API key or secret in the chat: signing in to fastn always happens in the browser.

## What your answer looks like

Everything below leads to one answer: does fastn fit this use case, and what does the user do next. The person reading is usually new to fastn and often not technical, so the answer is short, in plain words, and easy to act on. Write it in this shape:

1. **Verdict.** No heading or label: the answer simply starts with two sentences: first whether fastn fits and how it works for them in plain words, then whether you checked their fastn workspace or worked from fastn's docs. Then one sentence on what you found that makes you sure.
2. **How it works.** 3 to 5 bullets, one line each. Each starts with the fastn piece in bold, then the job it does in this use case.
3. **Steps to build it.** A numbered list in build order. Each step is one line and ends with its docs.fastn.ai link.
4. **Have ready.** One line on what to prepare, then one **Docs** line with the start page for this use case.
5. **Offer.** One line offering to build it with them. Then stop.

When something they need is missing, add one short line about it before the steps. Section 2.3 has the detail for each part.

Why this shape, and these limits:

- **About 250 words.** A new user decides from the verdict and acts on the steps. A long essay hides both, and the detail is more useful later, during the build.
- **No code, JSON, tables, database designs or sample data.** They belong in the build, after the user says yes; in a fit answer they make the setup look harder than it is. Action names in backticks are fine.
- **Plain words.** Say "each customer signs in to their own apps", not "multi-tenant OAuth". Leave out words like idempotency, payload or event-driven, or explain them in a few words.
- **No questions in or after the answer.** Ask your questions first, in a message of their own (Part 2). Ending on the offer tells the user exactly what to say next.
- **fastn holds the per-customer setup.** Each customer's connections and choices live in fastn (the widget and its settings), so do not tell the user to build their own routing or settings tables for it.
- **Only real names.** Name an action or unified entity only when you checked it exists. Working from the docs, say "for example" and describe it in plain words instead of inventing a name.
- **Standard records first.** Write to records every account has, such as a task, note or activity. A custom record type needs each customer to create it in their own app, so say so if you suggest one.
- **Links only from docs.fastn.ai and the GitHub files linked on this page.** Take docs links from the skill's docs-links table. Do not search the web for fastn: results include old pages that no longer exist and other companies' products, and they send the user the wrong way.

**If you cannot fetch web pages**, tell the user: "I can't open links. Please open the link in your browser, copy the whole page, and paste it here." Then follow the pasted text. Do the same for any other link below you need and cannot open.

## Part 1: Load the fastn skill

The fastn skill is a plain markdown file of instructions. Fetch it and follow it as instructions for the rest of this conversation, whatever AI tool you are. Fetch and read the fastn solution architect skill. It tells you how to work out whether fastn fits a use case, which fastn pieces to use, and how to set them up:

https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn-solution-architect/SKILL.md

Follow it for the rest of this conversation. When it points to one of its reference files, fetch that file only when you need it:

| Reference | Raw URL |
| --- | --- |
| `references/setup.md` (step-by-step setup for each piece) | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn-solution-architect/references/setup.md |
| `references/sdk.md` (Node.js and Python SDK, widget SDK) | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn-solution-architect/references/sdk.md |
| `references/edge-cases.md` (hard cases) | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn-solution-architect/references/edge-cases.md |
| `references/examples.md` (worked examples) | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn-solution-architect/references/examples.md |
| `references/meeting.md` (arranging a call with fastn) | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn-solution-architect/references/meeting.md |
| `references/docs-links.md` (the public docs page for each topic) | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn-solution-architect/references/docs-links.md |

If you cannot fetch the skill, work from the public docs at https://docs.fastn.ai and tell the user once that you are doing so.

## Part 2: Find out if fastn fits

Ask:

"What does your product do, and what should it do with your customers' apps?"

Before the answer, do not mention fastn accounts, sign-up or plans; account steps come only in Part 3, when the user wants to build.

After the opening question, ask at most two more before the answer, the workspace question included. For anything else, state your best guess in one line and answer.

Plan, limit and role facts come only from the workspace the user chose.

**The rules in this part override any answer format or order in the skill**, including its "Write it in this shape" section. Use the skill for what fastn can do and how to check it; use this part for which connection to use and how to write the answer.

### 2.1 Check the fit

First see whether fastn is already connected in this tool (fastn MCP tools are available).

* **fastn is connected:** use it to check the fit, following the skill: look up the connectors, actions, events and unified APIs the use case needs. Say in your answer that you checked their fastn workspace.
* **More than one fastn connection or workspace is connected:** before any fastn tool call (`whoami` included), name them as they appear in your tool list and ask which one to use for this. Never pick one yourself or call one to find out. Then use only that one, and report facts (connectors, events, plan) only from it. Until the user answers, work from the docs.
* **fastn is not connected:** answer from the skill and the docs only. Say plainly that the answer is based on fastn's docs, not on their account.

### 2.2 Use the docs, and link them

Always use the fastn docs, connected or not. Link every build step to its page from the table in the skill's `references/docs-links.md` (link in Part 1). Open a page only when you need a detail from it, such as a field, a limit or a trigger; https://docs.fastn.ai/llms.txt lists every page, and each is also available as markdown at its URL plus `.md`. Never make up a link, and do not use web search for fastn (see the top of this page for why).

### 2.3 Answer

Write the answer in the shape at the top of this page. The detail for each part:

1. **The opening.** No heading or label before it. Open with one or two short sentences that answer 'does fastn fit this?' directly. fastn is the subject and the verdict comes first: it fits, fits with one gap (name it), or does not fit today. In the same opening, tie the fit to what the user wants in plain words, and say whether you checked their fastn workspace or worked from fastn's docs. Keep the opening to two sentences at most: first the verdict with how it works for them, then the source. Anything else, such as which connection you used or what you still need to confirm, goes after it. Put no list of conditions in the opening. When something is really missing (the skill's section 9), still open with the verdict, name that one part, and say you have logged it with the fastn team. Say "Not today" only when it truly does not fit. Right after the opening, add one sentence on what you found in their workspace, or in the docs in definitions mode, that makes you sure. Write it in your own words. Sound sure when it fits; never oversell a gap. Never open with praise of the user's product, a restatement of their question, or a hedge such as 'it depends'.
   Good openings, as a guide to the shape only. Fill each part with this user's case in your own words; do not copy them:
   - 'Yes, fastn fits this well: <how it works for them, in one line>. <source>'
   - 'fastn is the right choice here: <the one setup that replaces many integrations>. <source>'
   - 'This is exactly what fastn is built for: <their flow in plain words>. <source>'
   - 'fastn fits this, with one gap: <what is missing>, and <how you would handle it>. <source>'
   - 'Not today: <what fastn can't do for this>. <source>'

   `<source>` is "I checked this in your fastn workspace." or "This is based on fastn's docs."

   Weak: opening with praise of the user's product instead of a verdict.
2. **How it works.** The fastn pieces for each need, 3 to 5 one-line bullets, each starting with the piece in bold. If a need has a gap, say the workaround in one line before the steps.
3. **The steps to build it, in order**, as a numbered list. Every step ends with its docs link; an answer without the links is not finished. For example: connect your product to fastn (your product as a connector), connectors and connections for the apps, the widget for customers to connect their apps, the trigger, the workflow, test and go live. Use only the steps this use case needs.
4. **Have ready and Docs.** One line on what to prepare: test accounts, a sample of the data, an https endpoint, whichever apply. Then one line with the docs start page for this use case and https://docs.fastn.ai.
5. **The offer.** If fastn is connected: "Do you want me to set it up and build it with you?" If not: "If you want to build it, connect fastn to this AI tool: I can walk you through it." Nothing comes after the offer: no further questions. Then follow Part 3 when the user wants to build.

## Part 3: Build it (needs a fastn account)

Only when the user wants to build. If fastn is already connected in this tool, go to 3.3. Otherwise start at 3.1.

### 3.1 Account

A fastn account is needed from here. Sign up free at https://app.fastn.dev/?signup=1 (the Launch plan is free: no card). Connecting an AI tool needs the **Developer** role or higher; an Owner or Admin can change roles in **Settings** → **People**.

### 3.2 Connect this AI tool to fastn

fastn connects to any AI tool that supports MCP (a standard way to give an AI tool access to an app). The fastn MCP server URL is `https://mcp.fastn.dev/fastn-workspace`.

**Every tool.** Add `https://mcp.fastn.dev/fastn-workspace` as a remote (HTTP) MCP server, with OAuth sign-in and no API key, then sign in to fastn in the browser when the tool asks. If you can change your own settings or run commands, do it yourself; otherwise give the user the steps.

Work out which tool you are running in. If you are not sure, ask the user one question: "Which AI tool are you using?" If it is listed below, use its exact steps. If not, use the step above and the guides for more tools: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway

**Claude Code.** Run:

```
claude mcp add --transport http --scope user fastn https://mcp.fastn.dev/fastn-workspace
```

Tell the user to start a new Claude Code session, type `/mcp`, pick **fastn** and choose **Authenticate**. Optional extra: the fastn plugin adds the fastn skills locally (`claude plugin marketplace add fastn-ai/fastn-claude-plugins`, then `claude plugin install connect@fastn`). Guide: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway/claude-code

**Claude app or claude.ai.** You cannot change these settings yourself. Give the user these steps:

1. Connect fastn: **Customize** → **Connectors** → **Add** → **Add custom connector**, name `Fastn`, URL `https://mcp.fastn.dev/fastn-workspace`. Keep the detected sign-in settings, click **Add**, then **Connect** and sign in.
2. Optional extra, the fastn plugin with the fastn skills: **Settings** → **Customize** → **Plugins** → **Add** → **Add marketplace**, paste `fastn-ai/fastn-claude-plugins`, select **Sync**, then turn on **fastn Connect**.

Guide: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway/claude

**Cursor.** Add fastn to `~/.cursor/mcp.json` (create it if missing, and keep any servers already in it):

```json
{
  "mcpServers": {
    "fastn": {
      "url": "https://mcp.fastn.dev/fastn-workspace"
    }
  }
}
```

Tell the user to open **Customize** → **MCPs** in Cursor and click **Authenticate** next to fastn. Guide: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway/cursor

**VS Code with GitHub Copilot.** Run:

```
code --add-mcp "{\"name\":\"fastn\",\"type\":\"http\",\"url\":\"https://mcp.fastn.dev/fastn-workspace\"}"
```

Tell the user to press **Cmd+Shift+P** (**Ctrl+Shift+P** on Windows and Linux), run **MCP: List Servers**, pick **fastn**, choose **Start Server** and allow the sign-in. fastn works in Copilot's **Agent** mode. Guide: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway/vscode

**Codex.** Run:

```
codex mcp add fastn --url https://mcp.fastn.dev/fastn-workspace
```

Codex opens the sign-in in the browser. If it does not, run `codex mcp login fastn`. Guide: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway/codex

**ChatGPT.** You cannot change these settings yourself. Give the user these steps: **Plugins** in the sidebar → **Add** → **Add custom MCP server**. Name `fastn`, Server URL `https://mcp.fastn.dev/fastn-workspace`, Authentication **OAuth**, tick **I understand and want to continue**, click **Create as a plugin**, then **Connect fastn** and sign in. Guide: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway/chatgpt

**Windsurf (now Devin Desktop), Cline, Antigravity, Lovable, Bolt.** Guides: https://docs.fastn.ai/build/building-from-your-ai-agent/mcp-gateway

**Gemini and any other tool.** Use the "Every tool" step above in the tool's MCP settings.

### 3.3 Build

Once the fastn tools are available, go back to the skill: check the user's workspace with the fastn tools, confirm the design, and build it step by step, following the steps from 2.3 and their docs pages. The skill names the fastn building skill each part needs. Fetch it when you reach that part:

| Skill | Use it for | Raw URL |
| --- | --- | --- |
| `gateway` | Read first, before your first fastn tool call | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/gateway/SKILL.md |
| `connector_builder` | Creating or fixing a connector for an app | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/connector_builder/SKILL.md |
| `integration_builder` | Building an integration: workflows, triggers, widget | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/integration_builder/SKILL.md |
| `unified_api` | One call that works across every app in a category | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/unified_api/SKILL.md |
| `workflow_verifier` | Checking a workflow works before it goes live | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/workflow_verifier/SKILL.md |
| `fastn_onboarding` | Walking through fastn's onboarding steps | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn_onboarding/SKILL.md |

A skill's own reference files sit next to it in `references/` in the same repository. For anything the skills do not cover, use the docs: https://docs.fastn.ai (every page is also available as markdown at its URL plus `.md`, and https://docs.fastn.ai/llms.txt lists them all).
