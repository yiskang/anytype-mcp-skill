# anytype-mcp skill

A [Claude Code skill](https://code.claude.com/docs/en/skills) that teaches Claude how to interact with your [Anytype](https://anytype.io) workspace using the [`anytype-mcp`](https://github.com/anyproto/anytype-mcp) server — querying and creating objects, managing types and properties, searching your knowledge base, and working with collections and GTD-style list views.

## Prerequisites

1. **Anytype desktop** running locally (the MCP server talks to its local API on port 31009)
2. **anytype-mcp installed** and configured in your Claude Code or Claude Desktop setup:
   ```bash
   npx -y @anyproto/anytype-mcp get-key   # follow prompts to generate an API key
   ```
3. **MCP server added** to your Claude config (`~/.claude/claude_desktop_config.json` or Claude Code MCP settings):
   ```json
   {
     "mcpServers": {
       "anytype": {
         "command": "npx",
         "args": ["-y", "@anyproto/anytype-mcp"],
         "env": {
           "OPENAPI_MCP_HEADERS": "{\"Authorization\":\"Bearer <YOUR_API_KEY>\", \"Anytype-Version\":\"2025-11-08\"}"
         }
       }
     }
   }
   ```

## Installation

### Claude Code

```bash
git clone https://github.com/yiskang/anytype-mcp-skill.git ~/.claude/skills/anytype-mcp
```

That's it. Start (or restart) a Claude Code session and the skill loads automatically.

> **Project-scoped install** (available only in one repo):
> ```bash
> git clone https://github.com/yiskang/anytype-mcp-skill.git ~/.claude/skills/anytype-mcp
> ```

### Claude Desktop

Download the ZIP from the [releases page](https://github.com/yiskang/anytype-mcp-skill/releases), go to Settings >> Capabilities >> Skills >> Customize >> Skills, click "+", click "Create a skill and "upload a skill" to upload the ZIP file, then enable the skill. Restart Claude Desktop to load it.

## Usage

Just talk to Claude naturally. The skill triggers automatically when your request involves Anytype:

```
Show me all tasks in my Inbox
```
```
Create a task called "Review Q2 report" with status Waiting
```
```
Search my Anytype space for notes about quarterly planning
```
```
How many objects do I have of type "book"?
```
```
List all views on my GTD task list
```
```
What types are available in my space?
```

On first use, Claude will discover your space ID with one tool call and hold it in context for the rest of the conversation. If you have multiple spaces, it will ask which one to use.

## What this skill covers

| Domain | Operations |
|---|---|
| Spaces & members | List spaces, get space, list/get members |
| Objects | List, get, create, update, delete, search (space + global) |
| Types | List, get, create |
| Properties | List, get, create |
| Tags | List, create |
| Templates | List, get |
| Collections & lists | Get views, get view objects, add/remove objects |

See [`references/operations.md`](references/operations.md) for the full tool reference with parameter details.

## File structure

```
anytype-mcp/
├── SKILL.md                  # Main skill instructions (loaded by Claude Code)
├── README.md                 # This file
└── references/
    └── operations.md         # Full MCP tool reference, loaded on demand
```

## Limitations

- **Body/markdown write** may not be supported depending on your Anytype API version — reading object content works, but writing back the full markdown body may fail
- **Members** are read-only (no add/remove via MCP)
- **Templates** are read-only (can use them when creating objects, but can't create new ones)

## Related

- [anytype-mcp](https://github.com/anyproto/anytype-mcp) — the MCP server this skill is built for
- [anytype-agents-skill](https://github.com/anyproto/anytype-agents-skill) — Anytype's official scripting-based skill (different approach, richer write operations)
- [Anytype API docs](https://developers.anytype.io) — underlying REST API reference

# License

This sample is licensed under the terms of the [MIT License](http://opensource.org/licenses/MIT).
Please see the [LICENSE](LICENSE) file for full details.

## Written by

Eason Kang [in/eason-kang-b4398492/](https://www.linkedin.com/in/eason-kang-b4398492), [Developer Advocacy and Support Team](http://aps.autodesk.com)