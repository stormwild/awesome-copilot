---
title: 'Sharing Sessions with Agent Host Protocol (AHP)'
description: 'Learn how to share Copilot CLI sessions across multiple terminals and remote machines using the Agent Host Protocol (--ahp).'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-15
estimatedReadingTime: '8 minutes'
tags:
  - copilot-cli
  - collaboration
  - sessions
  - remote
relatedArticles:
  - ./agents-and-subagents.md
  - ./copilot-configuration-basics.md
  - ./github-copilot-app.md
prerequisites:
  - GitHub Copilot CLI installed
  - Basic familiarity with Copilot CLI sessions
---

> **Early access**: Agent Host Protocol (AHP) is currently gated on the `AHP_CLIENT` feature flag and available to staff and early access users. The `--ahp` flag and `/ahp` slash commands are not yet available in general release builds.

The **Agent Host Protocol (AHP)** is a new capability in GitHub Copilot CLI that lets sessions live on a *host* rather than inside a single CLI process. Once a session is hosted, any number of terminals — on the same machine or across a network — can attach to it, watch it stream in real time, and steer it with their own prompts.

This changes two things: you can collaboratively watch and guide a single agent session from multiple terminals, and you can run sessions on remote machines (GitHub Codespaces, Mission Control cloud environments) without losing your local terminal workflow.

## The mental model

In a normal Copilot CLI session, the agent runs inside your terminal process. When you close the terminal, the session is gone. With AHP:

```
Your terminal  ─┐
                ├──► AHP Host (copilotd)  ──► Agent session
Colleague's    ─┘
terminal

  or

Local terminal ──► AHP Host on Codespace / Mission Control ──► Agent session
```

The **host** (`copilotd`) owns the session. Any **client** (a `copilot --ahp` terminal) attaches to the host and can send prompts, observe output, and steer running turns.

## Starting with AHP

Launch the CLI with the `--ahp` flag to attach to an AHP host. If no host is running, it starts one automatically:

```bash
copilot --ahp
```

Once attached, the Sessions tab in the sidebar shows every session on that host, including ones started by another terminal. Use `h` to move between hosts when multiple are available.

### Starting and managing hosts manually

You can also manage host daemons from inside an `--ahp` session:

```
/ahp start [port]        # start a new host daemon serving the current directory
/ahp stop <host>         # stop a named host
/ahp restart <host>      # restart a host (keeps serving the same workspace)
/ahp status              # show identity, health, and session count of the current host
```

`/ahp start` is also the fix when a healthy host refuses a new session with `permission denied` — it serves the current directory, so starting a new host scoped to your workspace clears the issue.

### Adding and switching between hosts

```
/ahp connect <url>           # add a host live (during a session)
/ahp hosts                   # list all registered hosts and their health
/ahp use <host>              # switch the active host from the timeline
```

You can also specify multiple hosts at startup:

```bash
copilot --ahp "wss://host1:8765,wss://host2:8765"
```

Or set `COPILOT_AHP_URL` to a comma-separated list of URLs so you don't have to type them each time.

## Connecting to remote machines

### GitHub Codespaces

The `/ahp codespace` command forwards a Codespace's `copilotd` port to your local machine and puts it in the Sessions tab source picker:

```
/ahp codespace my-codespace-name
```

You can use either the auto-generated Codespace name or the display name you gave it. Once connected, the Codespace appears in the Sessions tab marked `CS`, and you can create and attach to sessions on it just like a local host. The tunnel closes when you exit the CLI or run `/ahp stop <name>`.

> **Scope requirement**: `/ahp codespace` uses the `gh` CLI to forward the port. If your `gh` auth token is missing the `codespace` scope, the command shows you the exact `gh auth refresh` line to run.

### Mission Control cloud environments

The `/ahp cloud` command connects to a Mission Control environment:

```
/ahp cloud <environment-id>
```

Mission Control environments appear in the Sessions tab marked `CLOUD`. The environment wakes on connect and the CLI cannot start or stop it independently.

When you use `copilot --cloud` to provision a cloud session, the environment it provisions is also automatically placed in the Sessions tab source picker, so you can switch back to it or create additional sessions on it.

## Watching a session from multiple terminals

When another terminal is attached to the same session, the Sessions tab and sidebar show a `2 clients` badge (or more). Presence is announced when a client attaches and cleared when it disconnects.

All attached terminals see the same streaming output. Any terminal can steer the current turn:

| Action | Behavior |
|--------|----------|
| Type and press Enter | Steers the running turn (or sends a new prompt if idle) |
| Ctrl+Q | Queues the message for after the current turn finishes |
| Ctrl+C | Takes back a message that was steering the turn |

## Navigating sessions from the Sessions tab

In `--ahp` mode, the Sessions tab (`h` key) shows:

- A **source strip** at the top listing every connected host (bold = currently selected)
- Sessions from the selected host, sorted by activity (busy sessions first)
- Each session row shows the host it lives on and whether it is running, waiting for input, or idle
- `enter` — join (attach to) a highlighted session
- `n` — create a new session on the currently selected host
- Closing a row disposes the session on the host for all attached clients

The host itself appears at the top of the list with its daemon version and health. If the host stops responding, the host row turns red and a notice appears in the timeline.

## Security: redacting connection tokens

If you connect to a protected host using a connection token in the URL:

```bash
copilot --ahp "wss://host:8765?tkn=my-secret-token"
```

Copilot automatically redacts the `?tkn=...` query string from all output: `/ahp status`, session lists, error messages, and the connection notices. You can safely share transcripts without leaking tokens.

## Session directory behavior

An `--ahp` session runs in the directory you started the CLI from (when that directory is inside the host's workspace), not always at the host's workspace root. This means a daemon serving a parent directory (`~/projects/`) still starts your session in the right subdirectory (`~/projects/my-app`) when you connect from there.

## Auto-discovery

By default, `copilot --ahp` automatically discovers AHP daemons already running on your machine and adds them to the Sessions tab source picker. If you start a new daemon in another terminal while the CLI is open, it appears in the picker without needing to reconnect.

To disable auto-discovery:

```bash
COPILOT_AHP_DISCOVER=0 copilot --ahp
```

## Common questions

**Can I use `--ahp` without any remote machines?**

Yes. The simplest use case is two terminals on the same machine attaching to the same local daemon. One terminal runs the agent; the other watches and can steer if needed.

**What happens if the host goes away?**

The host row in the Sessions tab turns red and the disconnect is announced once in the timeline. Sessions that were running on the host are no longer reachable until the host comes back.

**Does AHP replace the Sessions tab for local sessions?**

No. Local sessions (not on a host) still work as before. `--ahp` is an opt-in mode that adds remote/shared hosting on top of the existing experience.

**Is AHP available in the GitHub Copilot app?**

The Copilot app has its own session management through its My Work view. AHP is a CLI-specific feature for sharing and remote-hosting CLI sessions.

## Further reading

- [Managing Sessions and Multiple Workflows](../copilot-configuration-basics/) — Background sessions, `/worktree`, and the Sessions tab
- [Agents and Subagents](../agents-and-subagents/) — Parallel task orchestration with `/fleet`
- [Getting Started with the GitHub Copilot app](../github-copilot-app/) — App-based parallel agent work

---
