# Agentic UI for Claude Code

Sends the notes you write on your running app with the Agentic UI Chrome extension straight into Claude Code. Claude fixes each one and marks its pin resolved. Nothing is installed in your app.

## Install

```sh
claude plugin marketplace add <owner>/<repo>
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

Channels are a Claude Code research preview, so the development flag and its confirmation are needed on every start for now. On Team and Enterprise plans an admin must turn channels on. Started with plain `claude`, the plugin still works, but notes wait until you ask Claude to "fix the pending annotations".

The bridge keeps its state in `.agentic-ui/` at the project root, a folder that git-ignores itself.

## Contents

- `server/bridge.mjs`: the local bridge (MCP server, channel and daemon) in one file
- `skills/fix-ui-annotations/`: how Claude handles the notes
- `commands/pair.md`: `/agentic-ui:pair`

`server/` and `skills/` are built from the Agentic UI source repository. Don't edit them here.
