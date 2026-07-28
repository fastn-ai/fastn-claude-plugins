---
name: gateway
description: How to work with the fastn integration gateway (connected as an MCP server by this plugin) - clear the gateway's required first read, pick the right fastn skill for the task, install it from its signed zip with one command, keep the installed copy in sync with the published version, and follow the local copy instead of re-fetching it. Use whenever a task touches an external app, connector, integration, workflow, sync, widget, or automation through fastn.
---

# fastn gateway

One governed endpoint fronting every app your organization has connected, which also serves fastn's library of skills for building integrations and running multi-tenant automations. Skills are published server-side, so discover the current set rather than assuming what exists. Auth, identity, and policy are handled by the gateway.

## Before anything else

**Your first call on this gateway is `skill {"slug":"gateway"}`.** Tool execution is REFUSED until this connection has read this playbook. The refusal names the fix, so a missed step is recoverable, but it wastes a round trip.

A bare `skill {}` listing does **not** clear the gate. Only a read that touches the `gateway` playbook does.

These stay open so you are never stuck: `skill`, `capture_feedback`, `manage_connections`, `search_tools`.

If your client opens a fresh session per request, the read may not stick. The read hands back a `_gw` token for that case: pass `_gw: "<token>"` alongside a tool's own arguments.

## Decide before you act

**Do not call app or platform tools to "look around" first.** Raw tool calls before a plan produce half-built work that has to be redone.

**This is enforced, not advice.** The gateway refuses your first app or platform call if this connection has not loaded a skill for the task, and the refusal lists the ones available. It fires once per connection, so a task that genuinely needs no skill just retries. Treat loading the matching skill as a blocking step, not a background task on your list: putting it on the list and doing the work first is exactly the failure this refusal exists to catch.

In order:

1. **Does this task need a skill?** Anything touching an integration, connector, workflow, sync, widget, automation, or external app: yes.
2. **Which skill?** Match the task with the routing table below. If unsure, `skill {}` and read the descriptions.
3. **Install it** (next section), then **read it**.
4. **Only then** start the work, following that skill's procedure.

| Task | Skill | You must read |
|---|---|---|
| Sync, integrate, migrate data between systems; build or modify a workflow; automate on an app event; schedule; expose a webhook; build a widget | `integration_builder` | Its SKILL.md **and** the reference files it names for the phase you are in |
| A connector, action, event, or auth method is missing or broken | `connector_builder` | Its SKILL.md **and** its references for the part you are changing |
| Just calling an app that already works | none | Call the app tool directly |

Reading the SKILL.md alone is not enough for these two. Both keep their per-phase procedure in `references/`, and skipping those is how a build ends up half-correct. Read each reference when you reach the phase that needs it, not all upfront.

Never tell the user the gateway is unavailable, and never hand-write an integration, until you have actually looked.

## Task 0: install checklist

Copy this into your task list and work it before the user's request. It is item #1, always.

```
Setup:
- [ ] 1. Read the gateway playbook (clears the gate)
- [ ] 2. Pick the skill(s) this task needs
- [ ] 3. Version check: skill {"slugs":["<slug>"]}
- [ ] 4. Installed at the same version? Skip to 5. Otherwise install now.
- [ ] 5. Claude Desktop / Cowork only: present the `.skill` file for EVERY skill loaded,
         including `gateway`, and tell the user to click Save skill
- [ ] 6. Read the installed SKILL.md, then its references as each phase needs them
```

Step 3 is one cheap call. An installed copy carries `<!-- fastn skill: <slug> v<N> -->`, so compare that tag against what the probe returns. Any version difference means reinstall, even if the body looks identical, because a reference file may have changed. The version number is the source of truth, not the visible text.

Do not skip step 3 and reinstall what you already have. Do not skip steps 3 to 5 and start the task with nothing installed.

## Install: one command

`downloadUrl` comes from `skill {}` or `skill {"slug":"x"}` and serves the complete skill as a zip (`<slug>/SKILL.md` plus `references/`). Plain GET, no auth header, **valid about 15 minutes**, so mint it immediately before you download. If a download 403s or reports an expired token, re-read for a fresh link rather than retrying the old one.

Work in ONE writable directory for the whole install and never mix it with another. **Do not use `/tmp`**: it is read-only in some sandboxes (Cowork among them), where every write fails with `Permission denied` and the install falls apart mid-way. Use your working folder, or the current directory. Set it once:

```bash
D="$(pwd)"                       # your working folder if you have one, else any writable dir
curl -fsSL "<downloadUrl>" -o "$D/<slug>.zip" && unzip -oq "$D/<slug>.zip" -d "$D"
ls "$D/<slug>" && ls "$D/<slug>/references"
```

**Keep the `-f` on curl.** Without it a 403 or an expired link is written into the `.zip` as an error page and curl still exits 0, so the unzip fails on garbage and everything after it improvises.

**That last `ls` is not optional.** It must show `SKILL.md` and a non-empty `references/`. If `references/` is missing you have a broken copy: do not continue, re-download. Splitting these steps across different directories, or unzipping with `-j`, flattens the folder into a loose `SKILL.md` and silently drops every reference.

Then **stamp the version** into `$D/<slug>/SKILL.md`, immediately after the closing `---` of the frontmatter, never above it or the file fails to load:

```
<!-- fastn skill: <slug> v<N> -->

> Step 0 - version sync (do this FIRST, every run): this file is tagged `<!-- fastn skill: <slug> v<N> -->`. Call `skill {"slugs": ["<slug>"]}` and compare. If the published version differs, reinstall from the fresh `downloadUrl` before doing anything else.
```

Now persist it. **Where a skill survives the session differs by client, and getting this wrong is the most common way an install silently does nothing.**

### Claude Desktop and Claude Cowork: build a `.skill` file

Skills live in the user's claude.ai account, not on disk. Re-zip the stamped folder with a `.skill` extension and present that file:

```bash
cd "$D" && zip -qr <slug>.skill <slug> && unzip -l <slug>.skill
```

**Read that listing before you present anything.** It must contain `<slug>/SKILL.md` and every `<slug>/references/*` file. A `.skill` holding one file is a broken install that looks finished: the skill loads next session and every procedure that opens a reference fails.

If `zip` is unavailable, build it with Python instead, then run the same check:

```bash
cd "$D" && python3 -c "import shutil;shutil.make_archive('<slug>.skill','zip','.','<slug>')" && mv <slug>.skill.zip <slug>.skill && unzip -l <slug>.skill
```

Presenting a `.skill` file renders a **Save skill** button. The user clicks it once and the skill is in their account: it persists across sessions, syncs across Claude surfaces, and carries `references/` with it. Saving a skill whose name already exists prompts the user to replace it, which is the update path here.

**Do both, in this order.** If your own tool list has a save-skill tool, call it too, passing the SKILL.md body: that puts the skill in the account immediately with nothing for the user to click. It may accept only a single body and drop `references/`, which is exactly why the `.skill` card is still required rather than optional. So: call your save-skill tool, **then** present the `.skill` file, then tell the user to click Save skill on it.

**This is mandatory, not a suggestion.** On these clients you have not installed anything until the `.skill` file is presented. Present one for **every** skill you loaded, the `gateway` playbook included, and tell the user explicitly to click Save skill on each card. Do not continue with the task and offer it afterwards, and never end a turn having read a skill without presenting its `.skill` file.

**Do not unzip into `.claude/skills` on these clients.** Unless the user has connected a working folder, that path is session scratch: it looks like a successful install and is gone next session. Even with a folder connected, the `.skill` file is what persists into the account, so present it either way.

### Claude Code, Copilot CLI, Codex CLI: move the folder into the skills directory (Check your identity first)

```bash
mkdir -p <skills-dir> && cp -R "$D/<slug>" <skills-dir>/
```

| Client | `<skills-dir>` |
|---|---|
| Claude Code | `.claude/skills` (project) or `~/.claude/skills` (personal) |
| Copilot CLI | `.claude/skills`, `.github/skills`, or `.agents/skills` (project), or `~/.copilot/skills` (personal). Then `/skills reload`, verify with `/skills info <slug>` |
| Codex CLI | `.codex/skills` (project) or `~/.codex/skills` (personal) |

Finally:

- **Read the SKILL.md you just installed, now.** A skill installed mid-session is usually not yet invocable by slug in that session, so use the file itself for this task.
- If you had to create the skills directory just now, it was not being watched. Say so: the client needs a restart.
- Tell the user the exact path it landed in.

### Rules

- **Never hand-write, paraphrase, or summarize a skill you could have downloaded.** The zip is the verbatim skill in one request and is effectively free. Rebuilding it from `skill` responses costs hundreds of times more tokens and drifts from the published text. Fall back to that only when the download genuinely cannot run, and then copy every file word for word.
- **A SKILL.md without its reference files is not installed.** The zip contains them; unzipping the whole thing is what satisfies this.
- A session temp or scratch directory is not persistence. Downloading to `/tmp` and stopping there means the skill is not installed.
- To update, overwrite with a fresh download and reload. Same command.

### Two things that are NOT installing

**A client's own save-skill tool** may accept only a single SKILL.md body, which **loses every reference file**. The `.skill` file carries them; that tool may not. Use it only for a skill with no references, and say what was dropped if you use it anyway.

**The gateway's own `save_skill`** PUBLISHES into your organization's shared library (owner/admin only). Calling it to "save" a skill you merely read republishes that skill org-wide.

Manual upload via **Customize > Skills > Add** also works if the user prefers it (the zip is already the right shape, and it needs Code Execution on under Settings > Capabilities), but the `.skill` button is one click and does the same thing.

## The `skill` tool

A single tool literally named `skill`. Your client namespaces it: `mcp__fastn__skill` on Claude, `fastn-skill` on Copilot CLI. The older `list_skills`, `load_skill`, and `open_skill_reference` names still route but are not listed, so searching for them finds nothing. **That does not mean the gateway is unavailable**: it may expose 200+ other tools alongside `skill`.

| Call | Does |
|---|---|
| `skill {}` | List every skill: slug, name, description, version, mode, `downloadUrl` |
| `skill {"slugs":["a","b"]}` | Cheap version probe |
| `skill {"slug":"x"}` | Read that skill's SKILL.md, its reference map, and a fresh `downloadUrl` |
| `skill {"slug":"x","ref":"<doc>"}` | Open one named reference document |
| `skill {"slug":"x","knownVersion":N,"toVersion":M}` | Per-file diff between two versions |
| `skill {"slug":"x","history":true}` | Full change history |

Reading never executes anything. To *run* a skill, call it by its slug like any other tool, but prefer the installed copy: running it re-ships the full instructions every call.

For a diff, re-fetch only the listed files (`~` changed, `+` added, `-` removed). If the diff is unavailable, reinstall in full.

## Task templates

Build the task list before the work, not during it. The published skill is authoritative: if its phases differ from these, follow the skill.

**`integration_builder`** (phases carry approval gates: stop and ask, do not build through them)

```
- [ ] 1. DISCOVER: connectors present and connection status
- [ ] 2. ANALYZE: entities on both sides
- [ ] 3. FEASIBILITY: required methods and events exist
- [ ] 4. PLAN: recommend approach, ask the business questions   [gate: user approves]
- [ ] 5. MAP: field mapping and config                          [gate: user approves]
- [ ] 6. TEST CASES                                             [gate: user approves]
- [ ] 7. BUILD: workflows, triggers, widget
- [ ] 8. VERIFY: execute and confirm the result
```

**`connector_builder`**

```
- [ ] 1. RESEARCH: API docs, auth model, endpoint list
- [ ] 2. CREATE: connector plus auth method
- [ ] 3. ACTIONS: full input and output schemas
- [ ] 4. CONNECT: an account to test against
- [ ] 5. TEST: execute every action live
- [ ] 6. EVENTS: wire events and triggers if needed
- [ ] 7. PROMOTE: to live
```

Expand any step into sub-steps when it has several parts (one per entity, one per action). Keep the list visible and check items off as you go.

## Using the app tools

- Rely on your current tool list, not prior assumptions.
- App tools are namespaced `app__action` (`slack__send_message`). Call the exact name shown.
- If `search_tools` is present, use it to find one, then `run_tool` with the name it returns. Otherwise the app tools are listed directly.
- Assume accounts are connected and just call the tool. If one is not, the call returns a connect link: hand it to the user. `manage_connections` lists what is connected and mints links.
- On a multi-step run, pass one short `_task` id on every call so the actions correlate.

## Feedback

If the user corrects how a skill behaves, call `capture_feedback` with the responsible skill's slug and the correction verbatim. That is the only channel reaching the skill owner's review queue.
