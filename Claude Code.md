---
author:
  - Dainish Jabeen
---
# Basics

## Commands

| Command             | What it does                                            | Example                             |
| ------------------- | ------------------------------------------------------- | ----------------------------------- |
| `claude`            | Start interactive mode                                  | `claude`                            |
| `claude "task"`     | Run a one-time task                                     | `claude "fix the build error"`      |
| `claude -p "query"` | Run one-off query, then exit                            | `claude -p "explain this function"` |
| `claude -c`         | Continue most recent conversation in current directory  | `claude -c`                         |
| `claude -r`         | Resume a previous conversation                          | `claude -r`                         |
| `/clear`            | Clear conversation history, good to reduce context fog. | `/clear`                            |
| `/help`             | Show available commands                                 | `/help`                             |
| `exit` or Ctrl+D    | Exit Claude Code                                        |                                     |
| /init               | Walks through creating a CLAUDE.md for your project     |                                     |
| /agents             | Configure custom sub agents                             |                                     |
| /doctor             | Any issues with installations                           |                                     |

## Git

Claude automatically has access to all your terminal commands including git, therefore can via natural language do all git commands.

```
commit my changes with a descriptive message (will determine message for you)
```

## Common workflows

Use these keywords:
- Refactor
- Write test
- Update
- Review

## Permission Modes

Shift+Tab to cycle through permission modes:

- Default: Asks before files are edited and commands used
- Auto-accept edits: Edit files without asking, permission for commands
- Plan mode: Read only tools, creates plan before execution
- Auto mode: Evals all actions with background safety checks.

*Can set trusted commands in settings.json file, to avoid asking.*

# Config/Extends

## CLAUDE.md

Project or system specific instructions, placed into the repo or claude dir that claude will read at the start of every session. Place to put conventions and common commands. 

- Keep under 200 lines
- Use skills for specific tasks or a path scoped rule.

## Subagents

When claude finds a task that matches the subagents description, it delegates to that subagent, which works independently and returns results. 

Subagents help you:

- **Preserve context** by keeping exploration and implementation out of your main conversation
- **Enforce constraints** by limiting which tools a subagent can use
- **Reuse configurations** across projects with user-level subagents
- **Specialize behavior** with focused system prompts for specific domains
- **Control costs** by routing tasks to faster, cheaper models like Haiku

**Main purpose i**

## Skills

Create a SKILL.md file with instructions, Claude will auto use them when relevant or can be used manually via /skill-name.

Placed into  `mkdir -p ~/.claude/skills/explain-code` the command here is /explain-code

Needs two parts a descriptions then list of instructions ie

```
---
name: explain-code
description: Explains code with visual diagrams and analogies. Use when explaining how code works, teaching about a codebase, or when the user asks "how does this work?"
---

When explaining code, always include:

1. **Start with an analogy**: Compare the code to something from everyday life
2. **Draw a diagram**: Use ASCII art to show the flow, structure, or relationships
3. **Walk through the code**: Explain step-by-step what happens
4. **Highlight a gotcha**: What's a common mistake or misconception?

Keep explanations conversational. For complex concepts, use multiple analogies.
```

## Hooks

Run shell commands automatically at precise moments in Claudes functioning (when it edits a file, finishes a task, needs input etc). Designed to be deterministic, prompt-based hooks or agent-based hooks also exist.

Placed into ``~/.claude/settings.json``


# Inner workings

Can be interrupted at any point and steered a different way.

Three step process on every prompt:
- Gather context
- Take Action
- Verify Results

**Claude code serves as the agentic harness (actionable) around claude providing the tools, context management and execution environment that turns LLM into a coding agent.**

## Access

When running claude in a directory, it has access to:
- All files in your directory
- The terminal (All commands available in the terminal)
- Your git state.
- Your Claude.md (stored in claudes directory) for each projects instructions.
- Auto memory: Learnings claude saves about your projects and patterns, stored in MEMORY.md
- Extensions in the claude directory (**skills, MCP, hooks, subagents and Claude in Chrome)

## Sessions

You are able to **rewind, resume and fork** sessions. Sessions are independent, each one starts without the previous conversation history can use the command to continue the previous. Conversations will carry across branch switches. Use git worktrees for separate sessions for each branch.

```
claude --continue --fork-session #Alows you to continue prev convo on a forked session
```

Multiple terminals running the same sessions will confuse the context, better to use forking !

## Context Window

```
/context # Sees what taking space in the context window.

/compact # Choose what parts of the context should be focused on when compacting can be added to CLAUDE.md
```

### Manage Context

- Skills load full context on demand
	- Can be set to be manual only aswell via [disable-model-invocation: true]
- Subagents get their own fresh context, when done they return a summary.


# Tips

- Context fog causes performance degradation. 

- Give something to verify against in any form

```
Implement func(), Test cases: 'meh' -> true, 'hem' -> false.
```

- Explore before implementing 
	- Treat it like a consultant that needs a well understood spec before starting work !!!