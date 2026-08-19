---
title: 'Working with Copilot Spaces'
description: 'Learn how to use Copilot Spaces to provide curated, project-specific context that grounds Copilot responses in your team''s actual code, documentation, and standards.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-19
estimatedReadingTime: '9 minutes'
tags:
  - spaces
  - context
  - knowledge-base
  - collaboration
relatedArticles:
  - ./understanding-copilot-context.md
  - ./understanding-mcp-servers.md
  - ./building-custom-agents.md
  - ./copilot-configuration-basics.md
prerequisites:
  - Basic understanding of GitHub Copilot
  - GitHub Copilot Pro, Pro+, Business, or Enterprise plan
---

Copilot Spaces are curated knowledge bases that bring project-specific context directly into your Copilot conversations. Instead of pasting documentation snippets or re-explaining your architecture every session, a Space packages your team's code, docs, and instructions in one place — and Copilot uses that context automatically.

Think of a Space as a shared briefing document that you load once, and Copilot draws on throughout the conversation.

## What Is a Copilot Space?

A Space is a named collection of resources owned by a user or organization:

- **Repositories** — selected files or whole repos
- **GitHub issues** — linked issues for live project tracking
- **Free-text documentation** — architecture notes, onboarding guides, standards
- **General instructions** — directives that guide Copilot's behavior within the Space

When you load a Space, Copilot gains access to all of those resources as grounding context. Answers and generated code are shaped by the real knowledge in your Space rather than Copilot's general training.

### Example Use Cases

| Scenario | What the Space Contains |
|----------|------------------------|
| Onboarding new team members | Architecture overview, coding standards, team conventions |
| Security compliance reviews | Internal policies, approved libraries, vulnerability checklists |
| Weekly project updates | Issue trackers, initiative summaries, report templates |
| Domain-specific development | API references, schema docs, proprietary framework guides |
| Cross-team standards | Shared patterns, style guides, architectural decision records |

## Discovering Available Spaces

### Using the MCP Server

If you have the GitHub MCP server configured, you can list available spaces from any Copilot CLI session:

```
What Copilot Spaces are available to me?
```

Copilot will call `mcp__github__list_copilot_spaces` and return the spaces you have access to, each with an owner and name.

### Using `gh api`

List your personal spaces:

```bash
gh api /users/{username}/copilot-spaces
```

List your organization's spaces:

```bash
gh api /orgs/{org}/copilot-spaces
```

## Loading a Space

Once you know the space name and owner, ask Copilot to load it:

```
Load the "Security Standards" space from the acme-corp organization
```

Or be more specific:

```
Using the Accessibility space, what are our WCAG compliance requirements?
```

Copilot will load the full Space context and use it to answer your question, following any custom instructions the Space defines.

### What Happens When a Space Is Loaded

1. Copilot retrieves the Space's content — documentation, code references, linked issues
2. The Space's `general_instructions` are applied as directives (treat them like custom instructions)
3. Copilot uses the Space's resources to ground all subsequent responses
4. If the Space references external resources (issues, dashboards, repos), Copilot can fetch them using other MCP tools

## Creating and Managing Spaces

Spaces are managed via the GitHub REST API using `gh api`. The MCP server provides read-only access; use `gh api` for any create, update, or delete operations.

### Creating a Space

```bash
gh api users/{username}/copilot-spaces \
  -X POST \
  -f name="My Project Space" \
  -f description="Context for the acme project" \
  -f general_instructions="Follow the team coding standards at docs/standards.md. Always prefer TypeScript. Use conventional commits."
```

For organization-owned spaces:

```bash
gh api orgs/{org}/copilot-spaces \
  -X POST \
  -f name="Engineering Standards" \
  -f description="Org-wide engineering standards and patterns" \
  -f visibility="private"
```

### Attaching Resources

After creating a space, attach content to it. Resources replace the entire list on each update, so include all existing resources when adding new ones:

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  --input - <<'EOF'
{
  "resources_attributes": [
    {
      "resource_type": "free_text",
      "metadata": {
        "name": "Architecture Overview",
        "text": "This project uses a microservices architecture with three main services..."
      }
    },
    {
      "resource_type": "github_issue",
      "metadata": {
        "repository_id": 12345,
        "number": 42
      }
    },
    {
      "resource_type": "github_file",
      "metadata": {
        "repository_id": 12345,
        "file_path": "docs/coding-standards.md"
      }
    }
  ]
}
EOF
```

**Resource types**:

| Type | Description |
|------|-------------|
| `free_text` | A named text block — for architecture notes, standards, documentation |
| `github_issue` | A linked GitHub issue — stays up to date as comments are added |
| `github_file` | A file from a GitHub repository — Copilot reads the file contents |

### Updating Space Instructions

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  -f general_instructions="Updated instructions here"
```

You can update name, description, instructions, and visibility together:

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  -f name="Updated Name" \
  -f description="Updated description" \
  -f general_instructions="Updated instructions" \
  -f visibility="private"
```

### Deleting a Space

```bash
gh api users/{username}/copilot-spaces/{number} -X DELETE
```

To find the space number, call the list endpoint first:

```bash
gh api /users/{username}/copilot-spaces
```

### Managing Collaborators

Add a collaborator to an organization-owned space:

```bash
gh api orgs/{org}/copilot-spaces/{number}/collaborators \
  -X POST \
  -f username="colleague" \
  -f role="reader"
```

Remove a collaborator:

```bash
gh api orgs/{org}/copilot-spaces/{number}/collaborators/colleague -X DELETE
```

## Authentication Requirements

Spaces operations require appropriate PAT scopes:

- **Read** (listing and loading spaces): `read:user` scope
- **Write** (creating, updating, deleting): `user` scope

If you get a 404 on write operations, refresh your token:

```bash
gh auth refresh -h github.com -s user
```

## Spaces as Workflow Engines

Beyond grounding answers in documentation, Spaces can encode entire workflows. A Space's `general_instructions` can define step-by-step processes that Copilot follows when the Space is loaded:

**Example: Weekly status update workflow**

```
## Weekly Update Instructions

When asked to write a weekly update:
1. Load the "Initiative Tracker" issue and summarize progress
2. List completed items from the milestone since last Monday
3. Identify blockers from open issues labeled "blocked"
4. Format the output using the template below

## Template
### Progress This Week
[list completed items]

### Blockers
[list any blockers]

### Next Week
[list planned items]
```

When a team member asks "write my weekly update using the PM space", Copilot follows this workflow, pulling from the linked issues and formatting the output exactly as specified.

## Spaces vs. Other Context Mechanisms

| Feature | Custom Instructions | Skills | Spaces |
|---------|--------------------|---------|---------| 
| Scope | File-pattern or always-on | Task-triggered | Explicitly loaded per session |
| Content | Text instructions only | Instructions + bundled assets | Docs, code, issues, instructions |
| Sharing | Repository-level | Repository or personal | User or organization-level |
| Live data | No | No | Yes (linked issues update automatically) |
| Best for | Coding standards, style | Recurring task workflows | Project knowledge bases, team onboarding |

## Tips and Best Practices

- **Space names are case-sensitive.** Use the exact name as returned by the list endpoint.
- **Spaces auto-update.** Linked GitHub issues and files reflect the latest content — your Space stays current as your project evolves.
- **Treat general instructions as directives.** If a Space says "always use TypeScript strict mode", Copilot will treat that as a requirement, not a suggestion.
- **Start focused.** A Space with 3–5 high-quality resources is more useful than one with 30 low-quality resources. Quality of context matters more than quantity.
- **Spaces work best alongside custom instructions.** Use instructions for universal standards (e.g., code style) and Spaces for project-specific knowledge (e.g., architecture docs for a specific service).
- **Use free-text resources for onboarding.** Write a concise "who we are, what we build, how we work" resource. New team members and Copilot both benefit.

## Current Status

> **Note:** The Copilot Spaces REST API is functional but may require the `copilot_spaces_api` feature flag in some environments. It is not yet fully documented in the public GitHub REST API docs. MCP read access via `mcp__github__list_copilot_spaces` and `mcp__github__get_copilot_space` is available with the GitHub MCP server.

## Next Steps

- **Add Context Depth**: [Understanding Copilot Context](../understanding-copilot-context/) — Learn how Copilot uses all available context signals
- **Connect Tools**: [Understanding MCP Servers](../understanding-mcp-servers/) — Configure the GitHub MCP server to use Spaces
- **Build Agents that Use Spaces**: [Building Custom Agents](../building-custom-agents/) — Create agents that load Spaces as part of their workflow
- **Explore the Skill**: Browse the [copilot-spaces skill](../../skills/) for ready-to-use Spaces integration patterns

---
