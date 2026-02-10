# AI Skills

Reusable AI skills for agents like [Claude Code](https://claude.ai) and [Cursor](https://cursor.com). Install them with a single command to enhance your AI workflows with structured, brand-aligned procedural knowledge.

## What Are AI Skills?

Skills are Markdown instruction files that give AI agents domain-specific expertise. Each skill contains structured prompts, frameworks, and guidelines that an agent reads and applies when performing a task. Think of them as downloadable playbooks your AI can follow on demand.

## Available Skills

| Skill | Path | Description |
| --- | --- | --- |
| **Brand Persona** | `brand-persona/SKILL.md` | The official Digital Speed brand voice, values, and communication guidelines. Use as a foundation for any brand-aligned content. |
| **Case Study Generator** | `case-study-generator/SKILL.md` | Generates case studies using the Problem &rarr; Solution &rarr; Outcome framework, written in the Digital Speed voice. |

## Installation

### Via the Skills CLI

```bash
npx skills add digitalspeed/ai-skills
```

This downloads the skill files into your project (typically in a `.skills/` directory), making them available to your AI agent.

## Usage

### Combining Skills

Skills are designed to be composable. The **Case Study Generator** references the **Brand Persona** automatically, but you can explicitly combine any skills to layer capabilities:

```text
Use @brand-persona.md to write a blog post about AI.
```

or 

```text
Apply the @case-study-generator.md skill using the @brand-persona.md persona
to transform this raw text into a case study: [paste your content]
```

### Example Using Cursor with the Claude Extension

1. Create a new project and install skills:

   ```bash
   mkdir blogs
   cd blogs
   npx skills add digitalspeed/ai-skills
   ```

2. Install the **ds-brand** skill when prompted.
3. Verify the file exists: `.cursor/skills/ds-brand/SKILL.md`
4. Open the Claude extension panel in Cursor and try:

   > Use @brand-persona.md to write a blog post about AI.

The agent will find the correct persona and generate a markdown blog post about AI.

## How Skills Work

1. **Discovery** -- The AI agent reads the YAML frontmatter and instructions at the top of each skill file.
2. **Triggering** -- When your prompt matches a skill's domain (e.g., "write a case study"), the agent loads the relevant instructions.
3. **Execution** -- The agent follows the skill's framework, tone guidelines, and step-by-step workflow to produce the output.

## Creating a New Skill

Each skill is a Markdown file with YAML frontmatter. Use this template:

```markdown
---
name: your-skill-name
description: A concise summary of what the skill does.
author: YourName
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
