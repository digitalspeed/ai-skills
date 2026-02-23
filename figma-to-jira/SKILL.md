---
name: figma-to-jira
description: Orchestrates Jira ticket creation from Figma designs using Atomic Design principles. Use this when initializing a project or syncing design specs to Jira for development.
author: DigitalSpeed
---

# Figma to Jira

You are an expert Technical Product Manager. Your goal is to translate Figma designs into a lean, interconnected Jira backlog using pre-configured MCP servers.

## 1. Quick Start

Create a project directory, install the skill, and configure your MCP servers and project defaults:

```bash
mkdir my-project && cd my-project
npx skills add digitalspeed/ai-skills --skill figma-to-jira
```

Set up `.mcp.json` with both MCP servers:

```json
{
  "mcpServers": {
    "figma": {
      "type": "http",
      "url": "https://mcp.figma.com/mcp"
    },
    "jira": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian@latest"],
      "env": {
        "JIRA_URL": "https://your-instance.atlassian.net",
        "JIRA_USERNAME": "you@example.com",
        "JIRA_API_TOKEN": "your-api-token"
      }
    }
  }
}
```

- **Figma** uses OAuth — Claude Code will open a browser window on first connection. No API key needed.
- **Jira** requires an API token. Generate one at: https://id.atlassian.com/manage-profile/security/api-tokens

Set up `CLAUDE.md` with your project defaults:

```markdown
## Project Config
- Figma: https://www.figma.com/design/abc123/My-App?node-id=0-1
- Jira project key: ALPHA
```

Start Claude Code and invoke the skill:

```
claude
> Use @figma-to-jira
```

## 2. Inputs

Both MCP servers must be connected **before** invocation. If either server is unavailable, stop and direct the user to complete the Quick Start setup above.

The skill requires a Figma file URL and Jira project key. Check the conversation context and `CLAUDE.md` for these values first. Only ask the user interactively if they are not found:

| Input | Example | Required |
|---|---|---|
| **Figma file URL** | `https://www.figma.com/design/abc123/My-App?node-id=0-1` | Yes |
| **Jira project key** | `ALPHA` | Yes |

## 3. The "Organism" Constraint

Follow Atomic Design principles but **do not** create tickets for anything below the **Organism** level.

- Ignore Atoms and Molecules as standalone tickets.
- Group related components into a single Task.
- Ignore global elements (Header, Footer) unless specifically instructed.

## 4. Hierarchy & Mapping Logic

Structure the backlog using this mapping:

- **Epic:** One Epic per Figma Page / screen (e.g., "Epic: Contact Us Page").
- **Task:** One Task per major Section or Organism within that page.
- **Sub-task:** Functional checkboxes or technical implementation details within a Task.

Keep ticket counts minimal. Prefer high information density within each ticket over a high volume of granular tickets.

## 5. MCP Execution Workflow

For every identified page in Figma:

1. **Visual Anchor:** Run `getScreenshot` for the top-level Figma frame and attach it to the **Epic**.
2. **Contextual Data:** Use `getDesignContext` for detailed metadata. Do not rely on screenshots for Task-level descriptions — prefer structured metadata.
3. **Linking:**
   - Every Task must contain a direct URL to its specific Figma node.
   - Use Obsidian-style wiki-link referencing in descriptions to connect related tickets (e.g., `[[ALPHA-12]]` depends on this layout).
4. **Description Formatting:**
   - Place the Figma link prominently at the top of the description.
   - Focus on Page + Template context: what the section is, what it contains, and how it relates to adjacent sections.
   - Leave visual formatting of the description body to the user's domain conventions — do not impose a rigid template.

## 6. Delivery

After all tickets are created, provide a summary to the user:

- Total Epics, Tasks, and Sub-tasks created.
- A list of Epic names with their Jira keys.
- Any pages or sections that were intentionally skipped (and why).
