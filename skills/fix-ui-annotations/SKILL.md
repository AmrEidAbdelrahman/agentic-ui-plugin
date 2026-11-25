---
name: fix-ui-annotations
description: Fix the UI annotations sent from the Agentic UI toolbar or Chrome extension, either through the local agentic-ui MCP server, as Jira tickets labeled agentic-ui, as Azure DevOps work items tagged agentic-ui, or as Linear, GitHub or GitLab issues labeled agentic-ui. Use when asked to fix pending annotations, UI notes, toolbar comments, a Jira key such as SHOP-142, an Azure DevOps work item such as #1234, a Linear issue such as ENG-12, a GitHub issue such as acme/shop#12, a GitLab issue such as web/shop#12, or the open agentic-ui tickets, or to keep fixing annotations as they come in.
---

<!-- Part of the agentic-ui Claude Code plugin. -->

The developer clicks elements in their running app, writes a comment, and sends it to you through the `agentic-ui` MCP server. Each annotation has the comment, the element (selector, visible text, test id, classes), and usually the React component hierarchy with a call site (where the component is rendered) and a source (where the clicked element's JSX is written).

## Fixing pending annotations

1. Call `list_pending`.
2. For each annotation:
   - Read the comment, then open the call site and source files. Decide which one the comment is about: a shared component (source) or the page that uses it (call site). Prefer the call site unless the comment is clearly about the component everywhere.
   - If there is no file location, or it looks stale, search the codebase for the element details (test id, aria-label, visible text, class names).
   - For a large or unclear change, call `acknowledge` first with a short plan or question.
   - Make the change.
   - Call `resolve` with a one-line summary the developer will see on the pin, for example "FilterPanel: Apply button is full width below 640px".
   - If no change is needed, or you can't do it, call `dismiss` with the reason.
3. Finish with a short list of what you changed.

A **recorded flow** is one annotation titled "Recorded flow" (ID `flow-…`): the developer's description of a bug that takes several steps, the errors, warnings and failed requests logged on the page while recording, and the path of the screen recording in the project (`.agentic-ui/flows/`). Its numbered steps are separate annotations that say "Flow step: N of recorded flow flow-…". Read the flow first and look for the cause in its problems; use the steps' elements and sources to find the code. If the flow has a "Replay:" line, it was recorded with the page's DOM too: call the `inspect_flow` tool with the flow's ID first for what was clicked and what changed on the page after each click, and with a step number for the page's HTML at that step. If you need to see a moment of the video, extract a frame with `ffmpeg` and read the image. Resolve the flow and its steps together.

## Fixing them as they come in

When asked to keep going, call `watch` after the pending list is empty. It returns new annotations as they arrive, or nothing after 60 seconds; fix what it returns, then call `watch` again. Stop when the developer says so.

In Claude Code, when the session runs the bridge as a channel, new annotations arrive on their own as a `<channel source="agentic-ui">` message, so there is no need to call `watch`. Call `acknowledge` before starting each one; if it says another session has it or it is already resolved, skip it.

If a note is ambiguous, or you need the developer to choose, call `ask_developer` with the note's ID instead of asking in the terminal or chat. The question appears on the note's pin in the browser, with any `options` as buttons, and the tool returns the answer. If it returns without one, move on; the answer comes later (see below). In Claude Code, permission prompts for the note you're working on also appear on its pin, so the developer can allow or deny from the browser.

The developer can also write to you from the extension's side panel. Answer with the `reply` tool, since the developer is reading the browser, not the terminal. In Claude Code, their messages and late answers arrive as `<channel source="agentic-ui" kind="message">` and `kind="answer"`. Agents without channels, such as Cursor, find them at the end of a tool result from the agentic-ui server, under "From the developer, in the browser:".

## Fixing Jira tickets

Testers and PMs file annotations from staging with the Chrome extension. Each becomes a Jira issue labeled `agentic-ui`. Read them with the Atlassian MCP server (`https://mcp.atlassian.com/v1/mcp/authv2`); if it isn't connected, ask the developer to add it and stop.

1. Find the issues:
   - For a key such as "fix SHOP-142", read that issue.
   - For "the open agentic-ui tickets in SHOP", search with JQL: `project = SHOP AND labels = agentic-ui AND statusCategory != Done ORDER BY created ASC`.
2. For each issue, take the annotation JSON from the collapsed "Agent context" section of the description. If the description says the JSON was shortened, read the `annotation.json` attachment instead. Look at `screenshot.webp` too if you can read attachments; the selected element is outlined.
3. Find the code:
   - Use `component.source` and `component.callSite` when they are there (localhost builds).
   - Otherwise search for the grep anchors: `anchors.testId` (data-testid), `anchors.ariaLabel`, `anchors.text` (visible text), then `anchors.classNames`. Narrow down with `page.route` or the URL path to find the page that renders it.
4. If `build.commit` is set, run `git log --oneline <commit>..HEAD -- <file>` for the file you will edit. If it changed since that commit, tell the developer before editing: the tester may have seen older code.
5. Make the change. Leave it on the current branch; don't commit, push or open a pull request unless the developer asks.
6. Comment on the issue with what you changed (files and a one-line summary).
7. Move the issue to the team's review status. The first time, ask the developer which status that is (for example "In Review") and save it in `.agentic-ui/jira.json` as `{ "reviewStatus": "In Review" }`; after that, read it from there. If the transition isn't available, say so and leave the status as it is.
8. Finish with a list of issues and what you changed in each.

## Fixing Azure DevOps work items

Annotations filed to Azure DevOps become work items tagged `agentic-ui`. Read them with Microsoft's Azure DevOps MCP server (`npx -y @azure-devops/mcp <organization>`); if it isn't connected, ask the developer to add it and stop.

1. Find the work items:
   - For a number such as "fix #1234" or "fix work item 1234", read that work item.
   - For "the open agentic-ui work items in Shop", run a WIQL query: `SELECT [System.Id] FROM WorkItems WHERE [System.TeamProject] = 'Shop' AND [System.Tags] CONTAINS 'agentic-ui' AND [System.State] NOT IN ('Done', 'Closed', 'Resolved', 'Removed') ORDER BY [System.CreatedDate] ASC`. State names depend on the project's process; adjust them if the query finds nothing.
2. For each work item, take the annotation JSON from the "Agent context (annotation.json)" block at the end of Repro Steps (Bugs) or Description (other types). If it says the JSON was too large, read the `annotation.json` attachment instead. Look at `screenshot.webp` too if you can read attachments; the selected element is outlined.
3. Find the code and check `build.commit` as in steps 3 and 4 for Jira above.
4. Make the change. Leave it on the current branch; don't commit, push or open a pull request unless the developer asks.
5. Comment on the work item with what you changed (files and a one-line summary).
6. Move the work item to the team's review state. The first time, ask the developer which state that is (for example "Resolved") and save it in `.agentic-ui/azure-devops.json` as `{ "reviewState": "Resolved" }`; after that, read it from there. If the state isn't allowed for the work item type, say so and leave it as it is.
7. Finish with a list of work items and what you changed in each.

## Fixing GitHub issues

Annotations filed to GitHub become issues labeled `agentic-ui`, without a screenshot (GitHub's API can't attach images). Read them with the GitHub CLI (`gh`), or GitHub's MCP server if it is connected. If neither works, ask the developer to run `gh auth login` and stop.

1. Find the issues:
   - For "fix acme/shop#12" or "fix GitHub issue 12", read it: `gh issue view 12 --repo acme/shop --json number,title,body`. A bare "#12" could also be an Azure DevOps work item or a GitLab issue: if more than one is set up, ask which.
   - For "the open agentic-ui issues", list them: `gh issue list --repo <owner/name> --label agentic-ui --state open --json number,title`. Without a repository named, use the current repository's.
2. For each issue, take the annotation JSON from the collapsed "Agent context (annotation.json)" block at the end of the body.
3. Find the code and check `build.commit` as in steps 3 and 4 for Jira above.
4. Make the change. Leave it on the current branch; don't commit, push or open a pull request unless the developer asks. If they do open one, add "Fixes #12" to its description so the issue closes when it merges.
5. Comment on the issue with what you changed (files and a one-line summary): `gh issue comment 12 --repo acme/shop --body "…"`. Leave the issue open for review unless the developer says to close it.
6. Finish with a list of issues and what you changed in each.

## Fixing Linear issues

Annotations filed to Linear become issues labeled `agentic-ui`, with the screenshot in the description. Read them with Linear's MCP server (`claude mcp add --transport http linear https://mcp.linear.app/mcp`). If it isn't connected, ask the developer to add it and stop.

1. Find the issues:
   - For "fix ENG-12", get that issue.
   - For "the open agentic-ui issues", list the issues labeled `agentic-ui` that aren't done or canceled, in the team the developer names (ask if there are several).
2. For each issue, take the annotation JSON from the "Agent context (annotation.json)" code block at the end of the description. Look at the screenshot too if you can open it; the selected element is outlined.
3. Find the code and check `build.commit` as in steps 3 and 4 for Jira above.
4. Make the change. Leave it on the current branch; don't commit, push or open a pull request unless the developer asks. If they do open one, put "ENG-12" in its branch name or title so Linear links it.
5. Comment on the issue with what you changed (files and a one-line summary). Leave its state as it is unless the developer says otherwise.
6. Finish with a list of issues and what you changed in each.

## Fixing GitLab issues

Annotations filed to GitLab become issues labeled `agentic-ui`, with the screenshot in the description. Read them with the GitLab CLI (`glab`). If it isn't signed in, ask the developer to run `glab auth login` (with `--hostname` for a self-managed GitLab) and stop.

1. Find the issues:
   - For "fix web/shop#12" or "fix GitLab issue 12", read it: `glab issue view 12 --repo web/shop --output json`. A bare "#12" could also be a GitHub issue or an Azure DevOps work item: if more than one is set up, ask which.
   - For "the open agentic-ui issues", list them: `glab issue list --repo <group/name> --label agentic-ui --output json`. Without a project named, use the current repository's.
2. For each issue, take the annotation JSON from the collapsed "Agent context (annotation.json)" block at the end of the description.
3. Find the code and check `build.commit` as in steps 3 and 4 for Jira above.
4. Make the change. Leave it on the current branch; don't commit, push or open a merge request unless the developer asks. If they do open one, add "Closes #12" to its description so the issue closes when it merges.
5. Comment on the issue with what you changed (files and a one-line summary): `glab issue note 12 --repo web/shop --message "…"`. Leave the issue open for review unless the developer says to close it.
6. Finish with a list of issues and what you changed in each.
