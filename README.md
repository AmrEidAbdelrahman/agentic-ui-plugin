# Agentic UI for Claude Code (and other coding agents)

Sends the notes you write on your running app with the Agentic UI Chrome extension straight into Claude Code. Claude fixes each one and marks its pin resolved. Nothing is installed in your app.

Codex, Grok Build, Gemini CLI, Cursor, GitHub Copilot and other MCP clients can use the same bridge: see [Other agents](#other-agents).

## Install

```sh
claude plugin marketplace add https://github.com/AmrEidAbdelrahman/agentic-ui-plugin.git
claude plugin install agentic-ui@agentic-ui
```

Needs Node.js 20 or later.

## Use

1. In your project, start Claude Code with the plugin as a channel:

   ```sh
   claude --dangerously-load-development-channels plugin:agentic-ui@agentic-ui
   ```

2. Run `/agentic-ui:pair`. In the extension's settings, under your localhost site, enter the code and click **Pair**. You pair once per project.
3. Annotate your app and click **Send to Claude Code**. The note appears in the session right away, and Claude fixes it.

For notes from the extension's accessibility check, Claude calls `check_accessibility` after its fix: the developer's tab on that page runs the same axe-core rule again and tells Claude which elements pass and which still fail.

Channels are a Claude Code research preview, so the development flag and its confirmation are needed on every start for now. On Team and Enterprise plans an admin must turn channels on. Started with plain `claude`, the plugin still works, but notes wait until you ask Claude to "fix the pending annotations".

The bridge keeps its state in `.agentic-ui/` at the project root, a folder that git-ignores itself.

## Other agents

Set the agent up once (Node.js 20 or later). This fetches the bridge from this repository, keeps a copy at `~/.agentic-ui/bin/bridge.mjs`, and adds it to the agent's MCP settings:

```sh
npx -y github:AmrEidAbdelrahman/agentic-ui-plugin setup codex
```

| Agent | Argument | Where it's added |
| --- | --- | --- |
| Codex (OpenAI) | `codex` | `~/.codex/config.toml` |
| Grok Build (xAI) | `grok` | `~/.grok/config.toml` |
| Gemini CLI | `gemini` | `~/.gemini/settings.json` |
| Cursor | `cursor` | the project's `.cursor/mcp.json` (run it in the project) |
| GitHub Copilot in VS Code | `copilot` | the project's `.vscode/mcp.json` (run it in the project) |

Cursor's and VS Code's files use `${userHome}` and `${workspaceFolder}`, so you can commit them; each teammate runs setup once to get their own copy of the bridge. Run setup again to update the bridge.

Then:

1. Restart the agent so it starts the `agentic-ui` server.
2. Ask the agent to "pair the extension" (or run `node ~/.agentic-ui/bin/bridge.mjs pair` in the project), and enter the code in the extension's settings, under your localhost site.
3. Annotate your app and click **Send to Codex** (the button is named after the agent connected). Ask the agent to "fix my UI notes".

These agents can't be sent notes on their own the way a Claude Code channel can: they pick them up with the `list_pending` and `watch` tools, so say "keep going" and they watch for more. What you write in the extension's side panel reaches them at the end of their next tool result. Their own tool approval prompts stay in the agent; they don't show on the pin.

Any other MCP client: add a stdio server that runs `node ~/.agentic-ui/bin/bridge.mjs mcp --agent mcp` in the project (after running setup for any agent once, so the file exists). It's named in the extension after the client's MCP handshake.

## Contents

- `server/bridge.mjs`: the local bridge (MCP server, channel and daemon) in one file
- `package.json`: makes `npx -y github:AmrEidAbdelrahman/agentic-ui-plugin` run the bridge, for other agents
- `skills/fix-ui-annotations/`: how Claude handles the notes
- `commands/pair.md`: `/agentic-ui:pair`

`server/`, `skills/` and `package.json` are built from the Agentic UI source repository. Don't edit them here.
