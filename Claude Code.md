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

# Inner workings

Can be interrupted at any point and steered a different way.

Three step process on every prompt:
- Gather context
- Take Action
- Verify Results

**Claude code serves as the agentic harness (actionable) around claude providing the tools, context management and execution environment that turns LLM into a coding agent.**

# Access

When running claude in a directory, it has access to:
- All files in your directory
- The terminal (All commands available in the terminal)
- Your git state.
- Your Claude.md (stored in claudes directory) for each projects instructions.
- Auto memory: Learnings claude saves about your projects and patterns, stored in MEMORY.md
- Extensions in the claude directory (**skills, MCP, hooks, subagents and Claude in Chrome)

# Sessions

You are able to **rewind, resume and fork** sessions. Sessions are independent, each one starts without the previous conversation history can use the command to continue the previous. Conversations will carry across branch switches. Use git worktrees for separate sessions for each branch.

```
claude --continue --fork-session #Alows you to continue prev convo on a forked session
```

Multiple terminals running the same sessions will confuse the context, better to use forking !

# Context Window

```
/context # Sees what taking space in the context window.

/compact # Choose what parts of the context should be focused on when compacting can be added to CLAUDE.md
```

## Manage Context

- Skills load full context on demand
	- Can be set to be manual only aswell via [disable-model-invocation: true]
- 


