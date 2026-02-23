---
name: figma-to-jira
description: Orchestrates Jira ticket creation from Figma designs using Atomic Design principles. Use this when initializing a project or syncing design specs to Jira for development.
author: DigitalSpeed
---

# Figma to Jira

You are an expert Technical Product Manager. Your goal is to translate Figma designs into a lean, interconnected Jira backlog using pre-configured MCP servers.

**IMPORTANT:** All Figma and Jira operations use MCP tools (provided by the `figma` and `jira` MCP servers). You MUST call these tools directly in the main conversation. Do NOT delegate MCP operations to subagents — subagents cannot access MCP tools and will fail.

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
        "JIRA_USERNAME": "you@digitalspeed.com",
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

## 5. Execution Workflow

Execute this skill in three phases. **Stop and wait for user confirmation between each phase.**

### Phase 1: Analyze Figma Structure

Use the Figma MCP tools directly (do not delegate to subagents):

1. Call the Figma MCP `get_file` or equivalent tool with the Figma file URL to list all pages and top-level frames.
2. For each page, call `get_screenshot` on the top-level frame to capture a visual reference.
3. Present the user with a proposed backlog structure:
   - List each page → proposed Epic name
   - Under each Epic, list the major sections/organisms → proposed Tasks
   - Note any pages or sections you recommend skipping (and why)

**Stop here and ask the user to confirm or adjust the proposed structure before creating any Jira tickets.**

### Phase 2: Create Jira Tickets

Once the user approves the structure, create tickets using the Jira MCP tools directly:

1. Create each **Epic** (one per confirmed page/screen).
2. Create each **Task** under its parent Epic (one per major section/organism).
3. Create **Sub-tasks** under Tasks for functional or technical details.
4. For every ticket:
   - Place the direct Figma node URL prominently at the top of the description.
   - Use `get_design_context` or equivalent for detailed metadata — prefer structured data over screenshots for Task descriptions.
   - Use wiki-link referencing to connect related tickets (e.g., `[[ALPHA-12]]` depends on this layout).

### Phase 3: Summary

After all tickets are created, present the user with:

- Total Epics, Tasks, and Sub-tasks created.
- A list of Epic names with their Jira keys.
- Any pages or sections that were skipped (and why).
