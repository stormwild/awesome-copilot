---
title: 'Using Copilot Spaces'
description: 'Learn how to use GitHub Copilot Spaces to create curated, shared knowledge bases that ground Copilot responses in your team''s actual code and documentation.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-05
estimatedReadingTime: '8 minutes'
tags:
  - spaces
  - context
  - collaboration
  - fundamentals
relatedArticles:
  - ./understanding-copilot-context.md
  - ./understanding-mcp-servers.md
  - ./copilot-configuration-basics.md
  - ./building-custom-agents.md
prerequisites:
  - Basic understanding of GitHub Copilot
  - GitHub Copilot Pro, Pro+, Business, or Enterprise plan
---

Copilot Spaces are curated, shared knowledge bases that bring project-specific context into your Copilot conversations. Instead of re-explaining your team's architecture, standards, and processes every session, you create a Space once and load it on demand. This article explains what Spaces are, how to use them, and how to manage them.

## What Are Copilot Spaces?

A **Copilot Space** is a named container of curated context — a collection of repositories, files, documentation, and custom instructions that you or your team assemble for a specific purpose.

When you load a Space in a conversation, Copilot receives all of that curated material as context and responds based on it. Spaces can hold:

| Resource Type | Description |
|--------------|-------------|
| `free_text` | Notes, guidelines, process docs, or any freeform text |
| `github_issue` | A specific GitHub issue (Copilot can follow live updates) |
| `github_file` | A specific file from a GitHub repository |

Spaces can be owned by an individual user or an organization, and their visibility can be set to `private` (owner/collaborators only) or `public`.

**Key characteristics**:
- Spaces auto-update as underlying repositories change, keeping context current
- Organizations can create shared Spaces for team-wide knowledge bases
- Collaborators can be managed per Space with configurable roles
- Spaces can encode both reference material and step-by-step workflows

### How Spaces differ from instructions and agents

- **Instructions** (`.github/copilot-instructions.md`) apply passively to every session in a repository — they're for persistent coding standards.
- **Agents** define a persona and toolset that reshapes how Copilot thinks throughout a session.
- **Spaces** are on-demand knowledge bases: loaded explicitly when you need a specific body of curated context. A single Space can serve many different sessions and be used with any agent.

## Discovering Available Spaces

To see what Spaces are available to you, use the `copilot-spaces` skill or ask Copilot directly:

> "What Copilot spaces are available?"

Copilot will call `mcp__github__list_copilot_spaces` and present the Spaces you can access. Each entry includes the Space name and its owner (a user login or organization name).

You can also filter by owner:

> "Show me spaces owned by our org"

## Loading a Space

Once you know the Space name, ask Copilot to load it:

> "Load the Security Guidelines space"
>
> "Using the Platform Architecture space, explain how our services communicate"

Copilot calls `mcp__github__get_copilot_space` with the owner and name, retrieves the full content, and uses it to inform its responses for the rest of the conversation.

> **Tip**: Space names are case-sensitive. If Copilot can't find a Space, ask it to list available Spaces first to confirm the exact name.

## Using a Space as a Workflow Engine

Spaces are not limited to reference material. You can encode step-by-step workflows inside a Space — templates, checklists, and structured processes — and then invoke them via natural language.

**Example: Weekly status update**

1. Create a Space called "PM Weekly Updates" with instructions and a template for your team's weekly report format
2. Attach relevant tracking issues and reference files
3. Each week, ask: *"Write my weekly update using the PM Weekly Updates space"*
4. Copilot loads the Space, follows the workflow, pulls data from the attached issues, and drafts the update in the right format

This pattern works for onboarding checklists, release processes, incident runbooks, architecture reviews, and any other repeatable workflow.

## Creating and Managing Spaces

The GitHub MCP server exposes read-only tools for Spaces. For creating, updating, and deleting Spaces, use the GitHub REST API via the `gh` CLI.

> **Note**: The Spaces API requires the `user` PAT scope for write operations. Run `gh auth refresh -h github.com -s user` if write calls return 404 errors.

### Create a Space

```bash
gh api users/{username}/copilot-spaces \
  -X POST \
  -f name="My Project Space" \
  -f description="Context for the main platform project" \
  -f general_instructions="Help me with platform architecture decisions following our ADR process." \
  -f visibility="private"
```

For organization Spaces, replace `users/{username}` with `orgs/{org}`.

### Update a Space

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  -f general_instructions="Updated instructions..." \
  -f name="New Name"
```

Find the `{number}` by listing your Spaces first:

```bash
gh api users/{username}/copilot-spaces | jq '.[] | {number: .number, name: .name}'
```

### Attach Resources

Resources are set as a complete list — include all existing resources plus any new ones when updating. To add a resource without losing existing ones, read the current list first and include all items:

```bash
gh api users/{username}/copilot-spaces/{number} \
  -X PUT \
  --input - <<'EOF'
{
  "resources_attributes": [
    {
      "resource_type": "free_text",
      "metadata": { "name": "Architecture Notes", "text": "Our services use event-driven architecture..." }
    },
    {
      "resource_type": "github_issue",
      "metadata": { "repository_id": 12345, "number": 42 }
    },
    {
      "resource_type": "github_file",
      "metadata": { "repository_id": 12345, "file_path": "docs/architecture.md" }
    }
  ]
}
EOF
```

To remove a resource, include it with `"_destroy": true`:

```json
{ "id": 123, "_destroy": true }
```

### Delete a Space

```bash
gh api users/{username}/copilot-spaces/{number} -X DELETE
```

### Manage Collaborators

Add collaborators to let others access or contribute to your Space:

```bash
# Add a collaborator
gh api users/{username}/copilot-spaces/{number}/collaborators \
  -X POST \
  -f login="collaborator-username" \
  -f role="reader"

# List collaborators
gh api users/{username}/copilot-spaces/{number}/collaborators

# Remove a collaborator
gh api users/{username}/copilot-spaces/{number}/collaborators/{collaborator_login} -X DELETE
```

## Practical Use Cases

### Team Onboarding Space

Create an org-level Space with:
- Architecture overview notes
- Key repositories and their responsibilities
- Engineering standards and ADRs
- Links to canonical issues tracking active initiatives

New team members load it on day one and ask questions grounded in your actual codebase.

### Incident Response Space

Create a Space for on-call engineers with:
- Runbooks for common failure modes
- Architecture diagrams as freeform text
- Links to monitoring issues and dashboards

During an incident, load the Space and ask: *"Our payment service is returning 503s — what do the runbooks say?"*

### Project Context Space

Track a major project in a Space:
- Attach the project planning issue
- Include key design documents as files
- Add the project spec as free text

During implementation, load the Space for context-aware code generation and review that reflects the project's specific requirements.

## Tips and Best Practices

- **Keep instructions focused**: The `general_instructions` field guides Copilot's behavior when the Space is loaded. Keep them specific and actionable.
- **Attach living resources**: Issues and files auto-update as the underlying content changes, so prefer them over copy-pasted free text when the source may change.
- **Use descriptive names**: Space names appear in Copilot's response when loading — clear names like "Platform Architecture" are more useful than "My Space".
- **Create org Spaces for shared standards**: Individual users own their own Spaces, but organization Spaces are the right home for team-wide knowledge bases.
- **Combine with skills**: The `copilot-spaces` skill from [Awesome Copilot](../../skills/) teaches agents how to discover and load Spaces automatically — install it for seamless integration.

## Learn More

- **Copilot Spaces skill**: [Install the `copilot-spaces` skill](../../skills/) from Awesome Copilot for automatic Space discovery and loading
- **Context management**: [Understanding Copilot Context](../understanding-copilot-context/) — How Copilot uses context and how Spaces fit in
- **Copilot Configuration**: [Copilot Configuration Basics](../copilot-configuration-basics/) — Instructions, agents, and skills that complement Spaces
- **MCP and tools**: [Understanding MCP Servers](../understanding-mcp-servers/) — How the GitHub MCP server powers Space access

---
