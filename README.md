# AI Skills

Reusable AI skills for agents like [Claude Code](https://claude.ai) and [Cursor](https://cursor.com). Install them with a single command to enhance your AI workflows with structured, brand-aligned procedural knowledge.

## What Are AI Skills?

Skills are Markdown instruction files that give AI agents domain-specific expertise. Each skill contains structured prompts, frameworks, and guidelines that an agent reads and applies when performing a task. Think of them as downloadable playbooks your AI can follow on demand.

## Available Skills

| Skill | Path | Description |
| --- | --- | --- |
| **Brand Guidelines** | `brand-guidelines/SKILL.md` | The DigitalSpeed visual identity and styling system — logo, color, typography, photography, and graphic system. Use when generating any design asset, UI, layout, or code that must conform to the DigitalSpeed brand styling. |
| **Brand Persona** | `brand-persona/SKILL.md` | The official Digital Speed brand voice, values, and communication guidelines. Use as a foundation for any brand-aligned content. |
| **Case Study Generator** | `case-study-generator/SKILL.md` | Generates case studies using the Problem &rarr; Solution &rarr; Outcome framework, written in the Digital Speed voice. |
| **Figma to Jira** | `figma-to-jira/SKILL.md` | Orchestrates Jira ticket creation from Figma designs using Atomic Design principles. Use when initializing a project or syncing design specs to Jira for development. |
| **License Checker** | `license-checker/SKILL.md` | Audits `node_modules` for package licenses and produces a compliance report. Use when reviewing open-source dependencies for license risk. |
| **Tutorial Generator** | `tutorial-generator/SKILL.md` | Generates step-by-step tutorials and how-to guides in the Digital Speed voice. Use when asked to write a tutorial, setup guide, or walkthrough. |

## Installation

### Via the Skills CLI

```bash
npx skills add digitalspeed/ai-skills
```

This downloads the skill files into your project `.agents/skills/` directory, making them available to your AI agent via symlinks, e.g.

```
.claude
└── skills
    ├── brand-guidelines -> ../../.agents/skills/brand-guidelines
    └── brand-persona -> ../../.agents/skills/brand-persona
    ...
```

## Usage

### Combining Skills

Skills are designed to be composable. The **Case Study Generator** references the **Brand Persona** automatically, but you can explicitly combine any skills to layer capabilities:

```text
Use @brand-persona to write a blog post about AI.
```

or 

```text
Apply the @case-study-generator skill using the @brand-persona persona to transform this raw text into a case study: [paste your content]
```

In practice, explicitly referencing is recommended because:

- It ensures the skill is actually loaded into the agent's context
- Automatic triggering depends on the agent's ability to discover and match skills, which varies by tool
- It removes ambiguity when you have multiple skills installed (e.g., brand-guidelines vs. brand-persona)

### Example Using Cursor with the Claude Extension

1. Create a new project and install skills:

   ```bash
   mkdir blogs
   cd blogs
   npx skills add digitalspeed/ai-skills --skill brand-persona
   ```

2. Install the skill when prompted.
3. Verify the file exists: `.cursor/skills/brand-persona/SKILL.md`
4. Open the Claude extension panel in Cursor and try:

   > Use @brand-persona to write a blog post about AI and write it to a markdown file.

The agent will find the correct persona and generate a markdown blog post about AI.

### Example Using Figma to Jira

1. Create a project directory and install the skill:

   ```bash
   mkdir ai-skills-<project> && cd ai-skills-<project>
   npx skills add digitalspeed/ai-skills --skill figma-to-jira
   ```

2. Set up `.mcp.json` with both MCP servers:

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
           "JIRA_URL": "https://digitalspeed.atlassian.net",
           "JIRA_USERNAME": "you@digitalspeed.co.uk",
           "JIRA_API_TOKEN": "your-api-token"
         }
       }
     }
   }
   ```

3. In Jira, create a new Scrum board: **Create project → Software development → Scrum**.

4. Set up `CLAUDE.md` with your project defaults:

   ```markdown
   ## Project Config
   - Figma: <insert figma url of project here>
   - Jira project key: <insert jira project key here>
   ```

5. Authenticate the Figma MCP server via OAuth:

   1. Open Claude Code in your project directory.
   2. Type `/mcp` — you should see the Figma and Jira servers listed.
   3. Select **figma → Authenticate** — this opens the browser for OAuth.
   4. Once both servers show as connected, proceed to the next step.

6. Add the following to `CLAUDE.md` to pre-approve all tools the skill uses so it runs without confirmation prompts:

   ```markdown
   ## Allowed Tools
   - mcp__figma__*
   - mcp__jira__*
   - Bash(python3:*)
   ```

7. Run Claude Code and invoke the skill:

   > Use @figma-to-jira

   Claude will read your Figma design and create the corresponding Jira epics, tasks, and tickets.

## How Skills Work

1. **Discovery** -- The AI agent reads the YAML frontmatter and instructions at the top of each skill file.
2. **Triggering** -- When your prompt matches a skill's domain (e.g., "write a case study"), the agent loads the relevant instructions.
3. **Execution** -- The agent follows the skill's framework, tone guidelines, and step-by-step workflow to produce the output.

## Creating a New Skill

Each skill is a Markdown file with YAML frontmatter. Use this template:

```markdown
---
name: your-skill-name
description: A concise summary of what the skill does and when to use it.
author: DigitalSpeed
---

# Skill Title

## Instructions
1. Define the step-by-step process the AI should follow.
2. Reference other skills by name if needed.
3. Specify tone, structure, and output format.

## Execution Workflow
- Step 1: Analyze input.
- Step 2: Apply the framework.
- Step 3: Refine output.
- Step 4: Return clean Markdown.
```

## Updating Skills

After making changes to skill files upstream, pull the latest versions into any project that uses them:

```bash
npx skills update
```
