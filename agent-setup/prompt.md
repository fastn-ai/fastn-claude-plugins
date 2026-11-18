# Set me up for fastn

These instructions are for any AI agent: ChatGPT, Claude, Cursor, GitHub Copilot, Codex, Gemini, Windsurf or another one. A person pasted this line into their AI tool:

```
Fetch and execute the instructions to set me up for fastn from https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/agent-setup/prompt.md
```

fastn lets a SaaS product offer integrations with the apps its customers use, set up by the customers themselves. Do the three parts below in order. Keep every message to the user short and in plain words. Never ask the user for a password, API key or secret in the chat: signing in to fastn always happens in the browser.

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

## Part 2: Find out if fastn fits (no account needed)

**Rules for this part. They override the skill's "check the live workspace" rule and its live mode until the user says they want to build (Part 3):**

* Do not call any fastn MCP tool, even if fastn MCP servers or connectors are already connected in this tool. That includes `whoami`, listing connectors, and anything else on a fastn server.
* Do not open, list, name or ask about fastn workspaces, orgs or connected servers. Do not mention that any are connected.
* Answer only from the skill (its definitions mode) and the public docs at https://docs.fastn.ai. Do not claim you checked the user's workspace.

Start with this, before any setup. Tell the user in one line: "You don't need a fastn account to find out whether fastn fits your use case." Then ask:

"What does your product do, and what should it do with your customers' apps?"

Once they answer, ask at most one or two short questions, and only if the answer truly depends on them. Otherwise state your assumptions in one line and answer. Use the skill to give the verdict (fits, fits with gaps, or does not fit today), the fastn pieces for each need, any gaps with a workaround, and the setup steps in order.

At the end, ask: "Do you want to build it? I can connect your AI tool to fastn and set it up with you."

## Part 3: Build it (needs a fastn account)

Only when the user wants to build. From here the fastn MCP server, workspaces and the skill's live checks are in play.

If fastn is already connected in this tool, ask which fastn workspace to use, then go to 3.3. Otherwise start at 3.1.

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

Once the fastn tools are available, go back to the skill: check the user's workspace with the fastn tools, confirm the design, and build it step by step. The skill names the fastn building skill each part needs. Fetch it when you reach that part:

| Skill | Use it for | Raw URL |
| --- | --- | --- |
| `gateway` | Read first, before your first fastn tool call | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/gateway/SKILL.md |
| `connector_builder` | Creating or fixing a connector for an app | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/connector_builder/SKILL.md |
| `integration_builder` | Building an integration: workflows, triggers, widget | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/integration_builder/SKILL.md |
| `unified_api` | One call that works across every app in a category | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/unified_api/SKILL.md |
| `workflow_verifier` | Checking a workflow works before it goes live | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/workflow_verifier/SKILL.md |
| `fastn_onboarding` | Walking through fastn's onboarding steps | https://raw.githubusercontent.com/fastn-ai/fastn-claude-plugins/main/plugins/connect/skills/fastn_onboarding/SKILL.md |

A skill's own reference files sit next to it in `references/` in the same repository. For anything the skills do not cover, use the docs: https://docs.fastn.ai (every page is also available as markdown at its URL plus `.md`, and https://docs.fastn.ai/llms.txt lists them all).
