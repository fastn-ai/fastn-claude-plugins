---
name: gateway
description: How to work with the fastn integration gateway (connected as an MCP server by this plugin) - clear the gateway's required first read, discover fastn's dynamically served library of skills with the `skill` tool, install a skill locally from its signed download link, keep the installed copy in sync with the published version, and follow the local copy instead of re-fetching it. Use whenever a task touches an external app, connector, integration, workflow, sync, or automation through fastn.
---

# fastn gateway

This plugin connects the fastn gateway as an MCP server: one governed endpoint fronting every app your organization has connected, which also **dynamically serves fastn's library of skills** for building integrations (connectors, workflows, syncs, widgets) and running governed, multi-tenant automations. Skills are published and updated server-side, so always discover the current set rather than assuming what exists. Authentication, identity, and policy are handled by the gateway - there is nothing to configure.

## First call, every time

**Your first call on this gateway is `skill {"slugs":["gateway"]}`.**

The gateway REFUSES every app tool (and `run_tool`) until this connection has read the `gateway` playbook. The refusal is a normal tool result that names the fix, so a missed step is recoverable, but it costs a wasted round trip. One probe avoids it and does two jobs at once: it clears the gate, and it tells you whether the copy you are reading right now is current.

- Published version matches the `<!-- fastn skill: gateway vN -->` tag in this file? You already have the rules. Continue.
- It differs, or this file carries no tag? Call `skill {"slug":"gateway"}`, follow what it returns, and reinstall per the install section below.

A bare `skill {}` listing does **not** clear the gate: it returns descriptions, not rules. Only a read that touches the `gateway` playbook does.

Some tools stay open so you are never stuck: `skill`, `capture_feedback`, `manage_connections`, and `search_tools`.

If your client opens a fresh session per request, a read may not stick. The playbook read hands back a `_gw` token for exactly that case: pass `_gw: "<token>"` alongside a tool's own arguments and the call goes through.

## The `skill` tool

Skill discovery runs through a **single tool literally named `skill`**. Your client namespaces it:

- Claude Code / Claude apps: `mcp__fastn__skill`
- GitHub Copilot CLI: `fastn-skill`

It is the only skill tool in your list. The older `list_skills`, `load_skill`, and `open_skill_reference` names still route if something calls them, but they are not listed, so searching your tool list for them finds nothing. **That does not mean the gateway is unavailable** - the gateway may expose 200+ other tools alongside `skill`.

Arguments select the action:

| Call | Does |
|---|---|
| `skill {}` | List every skill: slug, name, description, version, mode, `downloadUrl` |
| `skill {"slugs":["a","b"]}` | Cheap version probe for just those slugs |
| `skill {"slug":"x"}` | Read that skill's SKILL.md + its reference map + a fresh `downloadUrl` |
| `skill {"slug":"x","ref":"<doc>"}` | Open one named reference document |
| `skill {"slug":"x","knownVersion":N,"toVersion":M}` | Per-file diff between two versions |
| `skill {"slug":"x","history":true}` | Full change history |
| `skill {"withScore":true}` | List view plus quality scores and stale referenced tools (authoring) |

Reading never executes anything. To *run* a skill, call it by its slug like any other tool.

**On any task touching an integration, connector, workflow, sync, widget, automation, or external app, match the task to a skill slug before doing anything else.** Never tell the user the gateway is unavailable, and never hand-write an integration yourself, until you have actually looked.

## Using the app tools

- Treat the gateway as your first stop for any external app. Rely on your current tool list, not prior assumptions.
- App tools are namespaced `app__action` (`slack__send_message`, `github__list_issues`). Call the exact name shown.
- If a `search_tools` tool is present, use it to find one, then `run_tool` with the name it returns. If it is not present, the app tools are already listed directly in your tool list - use them there.
- Assume accounts are connected and just call the tool. If one is not, the call returns a connect link - hand that link to the user. `manage_connections` lists what is connected and mints connect links directly.
- On a multi-step run, pass one short `_task` id on every call so the actions correlate.

## Install and version-sync (mandatory)

Running a skill through its gateway tool re-ships the full instructions every call, which is expensive. A saved copy is cheap - but only if it is current. So, before using any skill, in this order:

1. **Do I already have it?** Check wherever your client keeps skills: your skills directories on a filesystem client, your saved skills on Claude Desktop or Cowork. An existing copy carries `<!-- fastn skill: <slug> v<N> -->`, so its version is readable without calling anything.
2. **What is published?** `skill {"slugs":["<slug>"]}` - one cheap call, and it returns the current version. (`skill {}` also works when you still need to pick which skill.)
3. **Same version?** Use the copy you already have and follow it. Do not re-read the skill through the gateway.
4. **Missing, or a different version?** Download and save it now, in this same turn, before continuing the task.

Any version difference triggers a re-download, even when the body looks unchanged - a reference file may have changed. The version number is the source of truth, not the visible text.

Never skip step 1 and re-download something you already have at the right version, and never skip steps 2 to 4 and start the user's task with no copy saved.

Installing is two separate steps, and **which client you are does not change the first one**.

#### Step 1: download the zip. Always, on every client.

`downloadUrl` (from `skill {}` or `skill {"slug":"x"}`) serves the complete skill as a zip - `<slug>/SKILL.md` plus `references/`. Plain GET, no auth header, valid ~15 minutes, multi-use; re-list for a fresh link.

```
curl -sSL "<downloadUrl>" -o /tmp/<slug>.zip
unzip -o /tmp/<slug>.zip -d /tmp/<slug>/
```

Any client that can run a command can do this, **including sandboxed app clients like Claude Cowork**. "My client has no persistent skills directory" is a reason to persist differently in step 2, never a reason to skip the download.

The zip is the whole skill in one request - verbatim by construction and effectively free. **Never hand-write, paraphrase, summarize, or reconstruct a skill you could have downloaded.** Rebuilding it from `skill {"slug":...}` + `{"ref":...}` responses costs hundreds of times more tokens and drifts from the published text, so treat that strictly as a fallback for when the download genuinely cannot run (no `downloadUrl`, or the request is blocked). If you do fall back, copy every file word for word.

Then **stamp the version** into the downloaded `SKILL.md`, immediately after the closing `---` of the YAML frontmatter - never above it, or the file fails to load:

```
<!-- fastn skill: <slug> v<N> -->

> Step 0 - version sync (do this FIRST, every run): this file is tagged `<!-- fastn skill: <slug> v<N> -->`. Call `skill {"slugs": ["<slug>"]}` and compare. If the published version differs, reinstall from the fresh `downloadUrl` before doing anything else.
```

#### Step 2: persist the downloaded files where YOUR client reads skills from

| Client | Persist to |
|---|---|
| Claude Code | `.claude/skills/` (project) or `~/.claude/skills/` (personal) |
| Copilot CLI | `.claude/skills/`, `.github/skills/`, or `.agents/skills/` (project), or `~/.copilot/skills/` (personal). Then `/skills reload`, verify with `/skills info <slug>` |
| Claude Desktop, Claude Cowork, claude.ai | Skills live in the user's claude.ai account, not on disk, and sync across Claude surfaces. **The zip you just downloaded is already in the required upload shape** (`<slug>/SKILL.md` at the zip root), so hand it to the user and tell them: **Customize > Skills > Add**, upload the zip. Needs Code Execution on under Settings > Capabilities. In Cowork you can also surface the downloaded files as a skill for the user to save |

On a client with a skills directory, moving the unzipped folder there IS the whole install - there is no separate save command:

```
cp -R /tmp/<slug>/<slug> .claude/skills/
```

Equivalently, unzip straight to the destination in step 1 (`unzip -o /tmp/<slug>.zip -d .claude/skills/`) and skip the copy.

**`save_skill` is not persistence.** It PUBLISHES a skill into your organization's shared fastn library (owner/admin only). Calling it to "save" a skill you merely read republishes that skill org-wide. Persisting a copy for yourself is always a client-side action.

A session temp or scratch directory is NOT persistence - it disappears with the session. Downloading to `/tmp` and stopping there means the skill is not installed.

Finally:

- **Read the SKILL.md you just downloaded, now.** A skill persisted mid-session is usually not yet invocable by slug in that session, so use the file itself for the current task. Installing is not a substitute for reading it.
- Claude Code picks up changes inside an EXISTING watched skills directory automatically. If you had to create that directory just now, it was not being watched, so tell the user to restart the client.
- Tell the user exactly where it landed. Never tell the user you cannot update a skill.

### Rules

- Fetching is not installing. Fetching gives you content for now; persisting keeps it for later runs. Both are required.
- The downloaded zip IS the verbatim skill - persisting its files unchanged satisfies the verbatim requirement. If you ever rebuild by hand instead (no `downloadUrl`), copy the body word-for-word: never paraphrase, summarize, reorder, or translate it. The version tag and Step 0 block are the only things you add.
- A SKILL.md persisted without all of its reference files is an incomplete install and counts as not installed. The zip already contains them; when rebuilding by hand, fetch each with `skill {"slug":"x","ref":"<doc>"}`.
- Put the install/update in your task list as item #1 and block the user's request on it, so it cannot be skipped. Do not start the user's task and install afterwards.
- To update an already-installed skill, overwrite it with a fresh download and reload. That is the update path on every client.

### Applying a diff instead of a full reinstall

```
skill {"slug":"digest","knownVersion":3,"toVersion":5}
"digest" v3 -> v5: 2 file(s) changed:
  ~ SKILL.md (changed)
  + reference:plan (added)
```

Re-fetch only the listed files: `~` changed, `+` added, `-` removed. If the diff is unavailable, do a full reinstall. Either way the result must be identical to the published version.

## Feedback

If the user corrects how a skill behaves, call `capture_feedback` with the responsible skill's slug and the correction verbatim. That is the only channel reaching the skill owner's review queue - do not post it anywhere else.
