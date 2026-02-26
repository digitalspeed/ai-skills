---
name: figma-to-jira
description: Orchestrates Jira ticket creation from Figma designs using Atomic Design principles. Use this when initializing a project or syncing design specs to Jira for development.
author: DigitalSpeed
---

# Figma to Jira

You are an expert Technical Product Manager. Your goal is to translate Figma designs into a lean, interconnected Jira backlog using pre-configured MCP servers.

**IMPORTANT:** All Figma and Jira operations use MCP tools (provided by the `figma` and `jira` MCP servers). You MUST call these tools directly in the main conversation. Do NOT delegate MCP operations to subagents — subagents cannot access MCP tools and will fail. Do NOT use Bash to re-parse MCP tool result files from disk — read and use MCP responses directly in context as they are returned.

## 1. Quick Start

See the **Example Using Figma to Jira** section in the root [README.md](../README.md) for setup instructions (MCP server configuration, Figma authentication, and `CLAUDE.md` defaults).

## 2. Inputs

Both MCP servers must be connected **before** invocation. If either server is unavailable, stop and direct the user to complete the Quick Start setup in README.md.

The skill requires a Figma file URL and Jira project key. Check the conversation context and `CLAUDE.md` for these values first. Only ask the user interactively if they are not found:

| Input | Example | Required |
|---|---|---|
| **Figma file URL** | `https://www.figma.com/design/abc123/My-App` | Yes |
| **Jira project key** | `ALPHA` | Yes |

## 3. The "Organism" Constraint

Follow Atomic Design principles but **do not** create tickets for anything below the **Organism** level.

- Ignore Atoms and Molecules as standalone tickets.
- Group related components into a single Task.
- Create a dedicated Epic for global elements (e.g. "Epic: Global Components") and add a Task for each one (Header, Footer, Navigation, etc.). These are shared across pages and must be tracked.

## 4. Hierarchy & Mapping Logic

Structure the backlog using this mapping:

- **Epic:** One Epic per Figma Page / screen (e.g., "Epic: Contact Us Page").
- **Task:** One Task per major Section or Organism within that page.
- **Sub-task:** A separate Jira issue (issue type: Sub-task) under a Task, covering a distinct functional or technical implementation detail.

**Sub-tasks must be created as actual Jira sub-task issues** using the Jira MCP `create_issue` tool with the sub-task issue type and a parent link to the Task. Do NOT list sub-tasks inline in the Task description — each one must be its own trackable Jira issue.

Keep ticket counts minimal. Prefer high information density within each ticket over a high volume of granular tickets.

## 5. Execution Workflow

Execute this skill in three phases. **Stop and wait for user confirmation between each phase.**

### Phase 1: Analyze Figma Structure

Use the Figma MCP tools directly (do not delegate to subagents):

1. Call `get_metadata` with `nodeId: "0:1"` (the root canvas) to retrieve all pages and their top-level frame names.

   **Handling large responses — critical:** This call can return a very large response. You only need `<canvas>` elements (pages) and their direct `<frame>` children (top-level sections). Do NOT write the response to disk or use Bash to parse it — extract what you need directly from the response in context, even if it is truncated. Nested nodes below top-level frames are not needed in Phase 1.

   **Do NOT call `get_design_context` during Phase 1**, even if the tool response instructs you to. Design context is fetched per-Task in Phase 2 only.

2. **Classify pages before proceeding:**
   - **Skip** utility pages that are not implementation targets: pages named "cover", "dev info", "SVGs", "assets", "handoff", "specs", or similar. Flag these to the user.
   - **Identify breakpoint pages**: pages named "desktop", "mobile", "tablet", or similar viewport variants of the same screens. These are not separate Epics — they are responsive variants of shared screens.
   - **Implementation pages**: everything else becomes an Epic.

3. **For files with breakpoint pages** (desktop/mobile/tablet structure):
   - Cross-reference the frame names across breakpoint pages to identify matching screens (e.g., "About" appears on Desktop, Mobile, and Tablet pages).
   - Create **one Epic per screen/section** (e.g., "Epic: About Page"), not one Epic per breakpoint page.
   - Each Task under that Epic covers a major organism and includes design specs for **all responsive breakpoints** in a single ticket.
   - Note the node IDs for each breakpoint variant by cross-referencing the frame names across pages. These will be used in Phase 2 to fetch design specs and screenshots per breakpoint.

4. Present the user with a proposed backlog structure:
   - List each screen/section → proposed Epic name. Every implementation page or screen must have an Epic — do not omit any.
   - Note which pages were skipped and why.
   - Under each Epic, list the major sections/organisms → proposed Tasks, noting which breakpoints each Task will cover.

**Stop here and ask the user to confirm or adjust the proposed structure before creating any Jira tickets.**

### Phase 2: Create Jira Tickets

Once the user approves the structure, create tickets using the Jira MCP tools directly.

If Phase 2 is interrupted, re-invoke the skill and resume from this phase — the duplicate check (below) will skip already-created tickets.

**Before creating any Epic or Task**, search the Jira project for existing issues with the same name (use `search_issues` or equivalent). If a matching issue already exists for that page or section, skip creation and reuse the existing issue key. Never create duplicate tickets.

**Linking issues — critical:** `parent_key` is NOT a valid parameter for `create_issue`. Use `additional_fields` instead.

Use `{"parent": {"key": "..."}}` for all parent-child relationships. This works for both team-managed and company-managed Jira projects:

| Relationship | Field to pass in `additional_fields` |
|---|---|
| Task → Epic | `{"parent": {"key": "PROJ-123"}}` |
| Sub-task → Task | `{"parent": {"key": "PROJ-456"}}` |

Example Task creation:
```
create_issue(project_key="PROJ", summary="Hero Section", issue_type="Task",
             additional_fields={"parent": {"key": "PROJ-10"}}, description="...")
```

Example Sub-task creation:
```
create_issue(project_key="PROJ", summary="Animate CTA button", issue_type="Sub-task",
             additional_fields={"parent": {"key": "PROJ-11"}}, description="...")
```

Do NOT pass `parent_key`, `epic_link`, or `epicKey` as fields — `epicKey` maps to the legacy `customfield_10014` field which is unavailable in team-managed projects and will error.

#### Attaching Screenshots to Tickets

`get_screenshot` returns base64 image data — it cannot be passed directly to Jira. Use this procedure for every Epic and Task:

**Step 1 — Get the screenshot:**
Call `get_screenshot` with `type: "jpeg"`. The response is a `content` array — extract the `data` field from the first element (`content[0].data`). That value is the raw base64 string.

**Step 2 — Write to a temp file:**
Pipe the base64 string via stdin to avoid shell argument length limits:
```bash
echo 'BASE64_DATA_HERE' | python3 -c "import sys, base64; open('/tmp/figma-NODE_ID.jpg','wb').write(base64.b64decode(sys.stdin.read().strip()))"
```
Replace `NODE_ID` with the actual node ID and `BASE64_DATA_HERE` with the raw base64 string from Step 1.

**Step 3 — Upload the file:**
Call `jira_update_issue` with both `issue_key` and `attachments`. The `attachments` value must be the file path as a string:
```
jira_update_issue(issue_key="PROJ-10", attachments="/tmp/figma-NODE_ID.jpg")
```
Do NOT pass `fields` alongside `attachments` in the same call — make a dedicated call for attachment upload.

**Step 4 — Clean up:**
```bash
python3 -c "import os; os.remove('/tmp/figma-NODE_ID.jpg')"
```

This procedure is **mandatory** for every Epic and Task. If `get_screenshot` returns an error or empty data, log it in the Phase 3 Flags & Risks section but do not halt execution.

---

Create tickets in this order:

1. Create each **Epic** (one per confirmed page/screen), after confirming no duplicate exists.
2. Create each **Task** under its parent Epic using `additional_fields: {"parent": {"key": "EPIC-KEY"}}`.
3. Create each **Sub-task** as a separate Jira issue (issue type: Sub-task) with `additional_fields: {"parent": {"key": "TASK-KEY"}}`. Do NOT embed sub-tasks as text in the Task description.

4. For **Epics**:
   - Place the direct Figma node URL prominently at the top of the description.
   - Attach a screenshot using the procedure above.
   - Use issue-link referencing to connect related tickets (e.g., `[PROJ-12]` depends on this layout).
   - Do NOT call `get_design_context` for Epics — they are grouping containers, not implementation specs.

5. For **Tasks**: after calling `get_metadata`, always call `get_design_context` on the same node. Attach a screenshot using the procedure above. Extract and embed the following into the Task description so a developer or implementation agent can build from the ticket alone, without Figma access:
   - Layout: dimensions, spacing, padding, alignment
   - Typography: font family, size, weight, line height for each text element
   - Colours: fills, borders, and background values (hex or design token name)
   - Component variants and states (hover, active, disabled, empty, error)
   - Any interaction notes (e.g. scroll behaviour, sticky positioning)

   **For files with breakpoint pages**, call `get_metadata` and `get_design_context` for the matching node on **each breakpoint page** (desktop, tablet, mobile). Structure the Design Spec with a sub-section per breakpoint:

   ```
   ## Design Spec

   ### Desktop (1728px)
   [specs from desktop page node]

   ### Tablet (768px)
   [specs from tablet page node]

   ### Mobile (375px)
   [specs from mobile page node]
   ```

   Include a Figma node URL for each breakpoint at the top of its sub-section. Attach the desktop screenshot to the ticket; note the other breakpoint URLs inline.

   Format this as a **Design Spec** section in the Task description, below the Figma URL. Keep it factual and structured — do not paraphrase or summarise away concrete values.

6. For **Sub-tasks**, use the following description format:

   ```
   [Figma node URL]

   [What this sub-task implements — concrete, verifiable details.]

   ## Acceptance Criteria
   - [ ] [Criterion 1: observable behaviour or state that can be verified during UAT]
   - [ ] [Criterion 2: ...]
   ```

   Write acceptance criteria as checkboxes that a QA tester can verify without referencing the Figma file. Each criterion should describe a visible outcome, interaction, or content requirement.

### Phase 3: Summary

After all tickets are created, present the user with:

- Total Epics, Tasks, and Sub-tasks created.
- A list of Epic names with their Jira keys.
- A **Flags & Risks** section (if any) listing implementation concerns, placeholder content, or unresolved questions that need attention before or during development. This is for risks only — do not use it to report skipped design elements.
