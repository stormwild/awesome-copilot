---
title: 'GitHub Copilot Terminology Glossary'
description: 'A quick reference guide defining common GitHub Copilot and platform-specific terms.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-19
estimatedReadingTime: '8 minutes'
tags:
  - glossary
  - terminology
  - reference
relatedArticles:
  - ./what-are-agents-skills-instructions.md
  - ./copilot-configuration-basics.md
---

New to GitHub Copilot customization? This glossary defines common terms you'll encounter while exploring agents, skills, instructions, and related concepts in the Awesome GitHub Copilot ecosystem.

Use this page as a quick reference when reading articles in the Learning Hub or browsing the repository.

---

## Core Concepts

### Agent

A specialized configuration file (`*.agent.md`) that defines a GitHub Copilot persona or assistant with specific expertise, tools, and behavior patterns. In products that support delegation, the agent is usually the primary coordinator or main session persona, while subagents handle narrower delegated tasks.

**When to use**: For recurring workflows that benefit from deep tooling integrations and persistent conversational context.

**Learn more**: [What are Agents, Skills, and Instructions](../what-are-agents-skills-instructions/)

---

### Subagent

A temporary, task-focused agent launched by another agent or orchestrator. A subagent usually gets a narrower prompt, its own isolated context window, and returns a summary back to the main agent instead of staying in the primary conversation.

**When to use**: For isolated research, parallel analysis, specialized review passes, or delegated implementation steps.

**Learn more**: [Agents and Subagents](../agents-and-subagents/)

---

### Built-in Tool

A native capability provided by GitHub Copilot without requiring additional configuration or MCP servers. Examples include code search, file editing, terminal command execution, and web search. Built-in tools are always available and don't require installation.

**Related terms**: [Tools](#tools), [MCP](#mcp-model-context-protocol)

---

### Chat Mode

**Deprecated terminology** - This term is no longer used. Use [Agent](#agent) instead.

Previously, "chat mode" was an alternative term for [Agent](#agent) that described how GitHub Copilot Chat could be transformed into domain-specific assistants. The ecosystem has standardized on "Agent" as the preferred terminology.

**See**: [Agent](#agent)

---

### Collection

**Note**: Collections are a concept specific to the Awesome GitHub Copilot repository and are not part of standard GitHub Copilot terminology.

A curated grouping of related skills, instructions, and agents organized around a specific theme or workflow. Collections are defined in YAML files (`*.collection.yml`) in the `collections/` directory and help users discover related customizations together.

**Example**: The "Awesome Copilot" collection bundles meta-skills for discovering and generating GitHub Copilot customizations.

**Learn more**: [What are Agents, Skills, and Instructions](../what-are-agents-skills-instructions/)

---

### Custom Agent

See [Agent](#agent). The term "custom" emphasizes that these are user-defined configurations rather than GitHub Copilot's default behavior. Custom agents can be created by anyone and shared via repositories like Awesome GitHub Copilot.

---

### Custom Instruction

See [Instruction](#instruction). The term "custom" emphasizes that these are user-defined rules rather than GitHub Copilot's built-in understanding. Custom instructions are particularly useful for codifying team-specific standards and architectural decisions.

---

## Configuration & Metadata

### Front Matter

YAML metadata placed at the beginning of Markdown files (between `---` delimiters) that provides structured information about the file and controls its behavior. In this repository, front matter typically includes fields like `name`, `description`, `mode`, `model`, `tools`, and `applyTo`.

The front matter is what controls:
- **Tool access**: Which built-in and MCP tools the customization can use
- **Model selection**: Which AI model powers the customization
- **Scope**: Where the customization applies (e.g., `applyTo` patterns for instructions)

**Note**: Not all fields are common across all customization types. Refer to the specific documentation for agents, skills, or instructions to see which fields apply to each type.

**Example**:
```yaml
---
name: 'React Component Generator'
description: 'Generate modern React components with TypeScript'
mode: 'agent'
tools: ['codebase']
---
```

**Used in**: Skills, agents, instructions, and Learning Hub articles.

---

### Handoff

A VS Code custom-agent frontmatter property (`handoffs`) that defines suggested transitions from one agent to another, often with a pre-filled follow-up prompt. Handoffs are useful for guided workflows such as research -> implementation or planning -> review.

**Important**: GitHub's [custom agent configuration reference](../building-custom-agents/#agent-configuration-reference) says `handoffs` are currently ignored for Copilot cloud agent on GitHub.com, so this concept is not portable across every Copilot surface.

**Learn more**: [Agents and Subagents](../agents-and-subagents/), [Building Custom Agents](../building-custom-agents/)

---

### AGENTS.md

An emerging industry standard file format for defining portable AI coding instructions that work across different AI coding tools (GitHub Copilot, Claude, Codex, and others). The `AGENTS.md` file, typically placed in a repository root or `.github/` directory, contains instructions for how AI assistants should interact with your codebase.

Unlike tool-specific customization files (`.agent.md`, `.prompt.md`, `.instructions.md`), `AGENTS.md` aims to provide a standardized, platform-agnostic way to define AI behavior that can be consumed by multiple tools.

**Key characteristics**:
- Platform-agnostic format for cross-tool compatibility
- Typically contains project context, coding standards, and architectural guidelines
- Located at repository root or in `.github/` directory

**Learn more**: [AGENTS.md Specification](https://agents.md/)

**Related terms**: [Instruction](#instruction), [Front Matter](#front-matter)

---

### Instruction

A configuration file (`*.instructions.md`) that provides persistent background context and coding standards that GitHub Copilot reads whenever working on matching files. Instructions contain style guides, framework-specific hints, and repository rules that help Copilot align with your engineering practices automatically.

**When to use**: For long-lived guidance that applies across many sessions, like coding standards or compliance requirements.

**Learn more**: [What are Agents, Skills, and Instructions](../what-are-agents-skills-instructions/), [Defining Custom Instructions](../defining-custom-instructions/)

---

## Skills & Interactions

### Persona

The identity, tone, and behavioral characteristics defined for an [Agent](#agent). A well-crafted persona helps GitHub Copilot respond consistently and appropriately for specific domains or expertise areas.

**Example**: A "Database Performance Expert" persona might prioritize query optimization and explain concepts using database-specific terminology.

**Related terms**: [Agent](#agent)

---

### Prompt

**Deprecated** — Prompts (`*.prompt.md`) were reusable chat templates that captured specific tasks or workflows, invoked using the `/` command in GitHub Copilot Chat. Prompts have been superseded by [Skills](#skill), which offer the same slash-command invocation plus agent discovery, bundled assets, and cross-platform portability.

If you have existing prompts, consider migrating them to skills. See [Creating Effective Skills](../creating-effective-skills/) for guidance.

**See**: [Skill](#skill)

---

### Skill

A self-contained folder containing a `SKILL.md` file and optional bundled assets (reference documents, templates, scripts) that packages a reusable capability for GitHub Copilot. Skills follow the open [Agent Skills specification](https://agentskills.io/home) and can be invoked by users via `/command` or discovered and invoked by agents automatically.

**Key advantages**:
- **Agent discovery**: Extended frontmatter lets agents find and invoke skills automatically
- **Bundled assets**: Reference files, templates, and scripts provide richer context
- **Cross-platform**: Portable across coding agent systems via the Agent Skills specification

**Example**: A `/generate-tests` skill might include a `SKILL.md` with testing instructions, a `references/test-patterns.md` with common patterns, and a `templates/test-template.ts` starter file.

**When to use**: For standardizing how Copilot responds to recurring tasks, especially when bundled resources improve quality.

**Learn more**: [What are Agents, Skills, and Instructions](../what-are-agents-skills-instructions/), [Creating Effective Skills](../creating-effective-skills/)

---

## Platform & Integration

### MCP (Model Context Protocol)

A standardized protocol for connecting AI assistants like GitHub Copilot to external data sources, tools, and services. MCP servers act as bridges, allowing Copilot to interact with APIs, databases, file systems, and other resources beyond its built-in capabilities.

**Example**: An MCP server might provide access to your company's internal documentation, AWS resources, or a specific database system.

**Learn more**: [Model Context Protocol](https://modelcontextprotocol.io/) | [MCP Specification](https://spec.modelcontextprotocol.io/) | [Understanding MCP Servers](../understanding-mcp-servers/)

**Related terms**: [Tools](#tools), [Built-in Tool](#built-in-tool)

---

### Hook

A shell command or script that runs automatically in response to lifecycle events during a Copilot agent session. Hooks are stored as JSON files in `.github/hooks/` and can trigger on events like session start/end, prompt submission, before/after tool use, and when errors occur. They provide deterministic automation—linting, formatting, governance scanning—that doesn't depend on the AI remembering to do it.

**Example**: A `postToolUse` hook that runs Prettier after the agent edits files, or a `preToolUse` hook that blocks dangerous shell commands.

**When to use**: For deterministic automation that must happen reliably, like formatting code, running linters, or auditing prompts for compliance.

**Learn more**: [Automating with Hooks](../automating-with-hooks/)

**Related terms**: [Agent](#agent), [Coding Agent](#coding-agent)

---

### Coding Agent

The autonomous GitHub Copilot agent that works on issues in a cloud environment without continuous human guidance. You assign an issue to Copilot, it spins up a dev environment, implements a solution, runs tests, and opens a pull request for review.

**Key characteristics**:
- Runs in an isolated cloud environment
- Uses your repository's instructions, agents, skills, and hooks
- Always produces a PR—it can't merge or deploy
- Supports iteration via PR comments

**When to use**: For well-defined tasks with clear acceptance criteria that can be completed autonomously.

**Learn more**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/)

**Related terms**: [Agent](#agent), [Hook](#hook)

---

### Plugin

An installable package that extends GitHub Copilot CLI with a bundled set of agents, skills, hooks, MCP server configurations, and LSP integrations. Plugins provide a way to distribute and share custom capabilities across projects and teams, with versioning, discovery, and one-command installation via marketplaces.

**Example**: Installing `database-data-management@awesome-copilot` to get a database specialist agent, migration skills, and schema validation hooks in a single command.

**When to use**: When you want to share a curated set of Copilot capabilities across multiple projects or team members, or when you want to install community-contributed tooling without manually copying files.

**Learn more**: [Installing and Using Plugins](../installing-and-using-plugins/)

**Related terms**: [Agent](#agent), [Skill](#skill), [Hook](#hook)

---

### Space (Copilot Space)

A curated knowledge base owned by a user or organization that grounds Copilot responses in project-specific context. A Space packages repositories, GitHub issues, free-text documentation, and custom instructions into a named collection. Loading a Space gives Copilot access to all of its resources for the duration of your conversation.

**Example**: A "Security Standards" space might contain your organization's compliance policies, approved library list, and vulnerability checklists — so Copilot answers security questions using your actual internal standards.

**Key characteristics**:
- Owned by a user or organization, with optional collaborator access
- Linked GitHub issues and files update automatically as the underlying content changes
- General instructions in a Space act as directives, guiding Copilot's behavior when the Space is active
- Accessible via the GitHub MCP server (read) or REST API (full CRUD)

**When to use**: For project-specific knowledge bases, team onboarding, domain-specific workflows, or any scenario where Copilot should draw on curated documentation rather than general knowledge.

**Learn more**: [Working with Copilot Spaces](../working-with-copilot-spaces/)

**Related terms**: [MCP](#mcp-model-context-protocol), [Instruction](#instruction), [Custom Agent](#custom-agent)

---

### Canvas

An interactive work surface inside the GitHub Copilot app where you and agents collaborate on a shared artifact. Instead of a linear chat thread, a canvas displays the actual work — a document, diagram, plan, or live output — that both you and the agent can edit and iterate on. Canvases can be evolved into reusable **Canvas Extensions** shared across your team.

**Example**: A `/create-canvas` command might produce a canvas showing a project plan. The agent updates sections as it works, and you can edit, approve, or redirect changes directly on the same surface.

**When to use**: For collaborative artifact creation, visual workflows, or any task where seeing the work in progress is more useful than reading a chat response.

**Learn more**: [Working with Canvas Extensions](../working-with-canvas-extensions/)

**Related terms**: [Coding Agent](#coding-agent), [Plugin](#plugin)

---

### Agent Merge

A Copilot app automation that carries pull requests through the full merge lifecycle autonomously. Once enabled, Agent Merge can monitor CI/CD pipelines, address failing tests or linting errors, track required reviewers, and automatically merge the PR when all conditions are met. The automation level is configurable — you can set it to just run CI, fix feedback, or go all the way to merging.

**Example**: After the coding agent opens a PR, Agent Merge monitors it, addresses review comments, waits for approvals, and merges when everything is green — without you having to check back manually.

**When to use**: When you want to eliminate the manual toil of shepherding PRs through CI and review cycles.

**Learn more**: [Getting Started with the GitHub Copilot app](../github-copilot-app/)

**Related terms**: [Coding Agent](#coding-agent), [Hook](#hook)

---

### Remote Control

A GitHub Copilot CLI feature that lets you connect to and steer a running coding agent session — either a local session you've made remotely accessible, or a cloud agent session — without waiting for it to finish. Remote control lets you observe the agent's progress in real time, send follow-up prompts, and redirect its work mid-task.

**Usage**:
```bash
copilot --remote     # start a remote-accessible session
/remote on           # enable remote control in an existing session
/remote off          # disable remote control
copilot --resume     # pick up any active remote session
```

**When to use**: For long-running agent tasks where you want to steer without waiting for a PR, or to provide mid-task clarification before the agent heads in the wrong direction.

**Learn more**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/)

**Related terms**: [Coding Agent](#coding-agent)

---

### Mission Control

The dashboard view on GitHub.com for tracking all active coding agent sessions across your repositories. Mission Control provides a centralized overview of what tasks are running, what's been completed, and what needs attention — without needing to check each repository separately.

**When to use**: When managing multiple parallel coding agent sessions and needing a single place to monitor their progress and outcomes.

**Learn more**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/)

**Related terms**: [Coding Agent](#coding-agent)

---

### Agentic Workflow

A markdown file that combines YAML frontmatter (triggers, permissions, safe outputs) with natural language instructions that a coding agent follows at runtime inside GitHub Actions. Agentic Workflows are compiled to `.lock.yml` files via the `gh aw` CLI and triggered by schedules, repository events, or slash commands.

**Example**: A `daily-issues-report.md` workflow that runs every weekday, reads open issues, writes a summary, and posts it as a new issue — without any manual intervention.

**Key characteristics**:
- Defined in a single `.md` file — no YAML Actions syntax required
- Natural language instructions are the workflow logic
- Use least-privilege permissions and safe outputs for security
- Compiled with `gh aw compile` before committing

**When to use**: For autonomous, event-driven repository automation that requires reasoning, summarization, or context-aware decisions beyond what static GitHub Actions steps can handle.

**Learn more**: [Agentic Workflows](../agentic-workflows/)

**Related terms**: [Coding Agent](#coding-agent), [Hook](#hook)

---

### Tools

Capabilities that GitHub Copilot can invoke to perform actions or retrieve information. Tools fall into two categories:

1. **Built-in tools**: Native capabilities like `codebase` (code search), `terminalCommand` (running commands), and `web` (web search)
2. **MCP tools**: External integrations provided by MCP servers (e.g., database queries, cloud resource management, or API calls)

Agents and skills can specify which tools they require or recommend in their front matter.

**Example front matter**:
```yaml
tools: ['codebase', 'terminalCommand', 'github']
```

**Related terms**: [MCP](#mcp-model-context-protocol), [Built-in Tool](#built-in-tool), [Agent](#agent)

---

**Have a term you'd like to see added?** Contributions are welcome! See our [Contributing Guidelines](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md) for how to suggest additions to this glossary.
