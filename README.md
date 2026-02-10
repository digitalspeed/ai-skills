# digitalspeed/ai-skills

Reusable AI skills for agents like [Claude Code](https://claude.ai) and [Cursor](https://cursor.com). Install them with a single command to enhance your AI workflows with structured, brand-aligned procedural knowledge.

## What Are AI Skills?

Skills are Markdown instruction files that give AI agents domain-specific expertise. Each skill contains structured prompts, frameworks, and guidelines that an agent reads and applies when performing a task. Think of them as downloadable playbooks your AI can follow on demand.

## Available Skills

| Skill | File | Description |
| --- | --- | --- |
| **Brand Persona** | `brand-persona.md` | The official Digital Speed brand voice, values, and communication guidelines. Use as a foundation for any brand-aligned content. |
| **Case Study Generator** | `case-study-generator.md` | Generates case studies using the Problem &rarr; Solution &rarr; Outcome framework, written in the Digital Speed voice. |

## Installation

### Via the Skills CLI

```bash
npx skills add digitalspeed/ai-skills
```

This downloads the skill files into your project (typically in a `.skills/` directory), making them available to your AI agent.

### Manual Installation

Clone the repository and copy the skill files you need into your project's skills directory:

```bash
git clone https://github.com/digitalspeed/ai-skills.git
cp ai-skills/brand-persona.md .claude/skills/
cp ai-skills/case-study-generator.md .claude/skills/
```

Adjust the target path to match your tool's convention (e.g., `.cursor/rules/` for Cursor).

## Usage

### Claude Code

Reference installed skills using slash commands or by mentioning the skill name in your prompt:

```
Apply the case-study-generator skill using the ds-brand persona to transform
these raw notes into a case study: [paste your content]
```

### Cursor

Use the `@` symbol to inject skill files as context:

```
Apply the @case-study-generator.md skill using the @brand-persona.md persona
to transform this raw text into a case study: [paste your content]
```

### Combining Skills

Skills are designed to be composable. The **Case Study Generator** references the **Brand Persona** automatically, but you can explicitly combine any skills to layer capabilities:

```
Use @brand-persona.md to write a blog post about our new SDK release.
```

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

### Naming Convention

Follow the `<namespace>/<skill-name>` pattern when registering with [skills.sh](https://skills.sh):

```
digitalspeed/case-study-generator
digitalspeed/brand-persona
```

## Publishing to skills.sh

To make your skills publicly discoverable:

1. Push your skill files to GitHub.
2. Go to [skills.sh](https://skills.sh) and sign in with GitHub.
3. Submit your repository URL (`https://github.com/digitalspeed/ai-skills`).

The platform indexes your repository and makes each skill searchable and installable.

## Updating Skills

After making changes to skill files upstream, pull the latest versions into any project that uses them:

```bash
npx skills update
```

## Contributing

1. Fork this repository.
2. Create a new branch for your skill (`git checkout -b skill/my-new-skill`).
3. Add your skill file to the repository root following the template above.
4. Update the **Available Skills** table in this README.
5. Open a pull request with a description of the skill and example usage.

## License

MIT
