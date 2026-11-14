---
name: fix-ui-annotations
description: Fix the UI annotations sent from the Agentic UI toolbar or Chrome extension, either through the local agentic-ui MCP server or as Jira tickets labeled agentic-ui. Use when asked to fix pending annotations, UI notes, toolbar comments, a Jira key such as SHOP-142, or the open agentic-ui tickets, or to keep fixing annotations as they come in.
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

## Fixing them as they come in

When asked to keep going, call `watch` after the pending list is empty. It returns new annotations as they arrive, or nothing after 60 seconds; fix what it returns, then call `watch` again. Stop when the developer says so.

When the session runs the bridge as a channel, new annotations arrive on their own as a `<channel source="agentic-ui">` message, so there is no need to call `watch`. Call `acknowledge` before starting each one; if it says another session has it or it is already resolved, skip it.

If a note is ambiguous, or you need the developer to choose, call `ask_developer` with the note's ID instead of asking in the terminal. The question appears on the note's pin in the browser, with any `options` as buttons, and the tool returns the answer. If it returns without one, move on; the answer arrives later as a `<channel source="agentic-ui" kind="answer">` message. Permission prompts for the note you're working on also appear on its pin, so the developer can allow or deny from the browser.

The developer can also write to you from the extension's side panel (its Claude Code tab). Those messages arrive as `<channel source="agentic-ui" kind="message">`; answer with the `reply` tool, since the developer is reading the browser, not the terminal. Every permission prompt also shows in that tab.

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
