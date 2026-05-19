# Claude

Personal tips for working with Claude Code effectively.

---

## Workflow

**Start sessions with context**
Open relevant files in the IDE before prompting — Claude sees what's open and uses it.

Start with `/init` to get Claude to look at entire codebase and get a sense. It will put its findings in a project-level `CLAUDE.md`.

### `CLAUDE.md` files
* `CLAUDE.md` - project-level, committed to repo/shared with other engineers
* `CLAUDE.local.md` - not shared/committed, but project-level
* `~/.claude/CLAUDE.md` - used with all projects on your machine

To write to any of the above `CLAUDE.md` files, type something in the Claude prompt starting with `#`.

**Give Claude a headstart on where to look**
Include relevant files with `@`.

**Ask for follow-ups before implementing**
For non-trivial tasks, ask Claude to surface clarifying questions before writing code. Saves back-and-forth mid-implementation.

**End sessions with a summary**
Ask Claude to summarize what changed and why before closing. Useful for CHANGELOG entries and picking up next session.

---

## Commands

| Command | What it does |
|---|---|
| `/memory` | View and edit persistent memory |
| `/ide` | for getting changes to reappear in the VS Code IDE |
| `CTRL+V` | Paste a screenshot in the command line (note: this is NOT `CMD`!) |
| `SHIFT+TAB SHIFT+TAB` | Prompt Claude to use plan mode in the next step -- good for complex tasks |
| `Think`, `Think more`, `Think a lot`, `Think longer`, `Ultrathink` | Trigger thinking mode, with progressively more token budget for more complex tasks |
---

### Custom Commands

* Make a `.claude/commands/` directory in your project folder
* To create a custom command called e.g. `/custom-command`, create a `custom-command.md` file in the above directory.
* The body of the Markdown file should include what the custom command does.
* Custom commands can accept arguments that are used in the body of the command with the `$ARGUMENTS` placeholder.

## Tasks

Tasks help Claude track progress on multi-step work within a conversation. Claude creates and updates them automatically, but you can also prompt it to use them explicitly.

**When they're useful:**
* Implementations spanning multiple files or steps
* Long sessions where you want to see what's done vs. in-progress
* Any time you want Claude to be explicit about its plan before executing

**How to prompt for them:**
```
Break this into tasks before starting.
Track your progress with tasks as you go.
```

Tasks are stored in `~/.claude/tasks` — you can use this to build additional utilities on top of them.

---

## `claude -p` (Non-interactive / Pipe Mode)

Use `claude -p` when you want Claude to run a single prompt and return output — no back-and-forth, no UI.

**When to use it:**
* Scripting and automation — pipe Claude into shell workflows
* One-shot tasks where you don't need a session (summarize a file, generate a commit message, etc.)
* CI/CD pipelines or pre-commit hooks
* Chaining with other tools (`cat error.log | claude -p "what's wrong here?"`)

```bash
claude -p "summarize this" < README.md
git diff | claude -p "write a commit message for this diff"
claude -p "generate a .gitignore for a Python project" > .gitignore
```

Use the interactive `claude` (no flag) when the task needs iteration, file edits, or tool use across multiple steps.

---

## Prompting

**Be explicit about scope** — "only change X" prevents unwanted refactoring.

**Reject and redirect** — if an approach is wrong, say so immediately rather than letting it continue building on a bad foundation.

**Interrupt + Memory files** - if you see Claude making a mistake repeatedly, a good pattern is to interrupt (`Esc`) followed by adding to memory (`#`). 

**Use live docs for new features**
Claude Code's training data lags behind its documentation. Asking about recent features (hooks, MCP, etc.) may produce an outdated or hallucinated answer.

Anthropic maintains a docs map at `code.claude.com/docs/en/claude_code_docs_map`. Append `.md` to get raw markdown Claude can read directly — then paste that URL alongside your question:

```
https://code.claude.com/docs/en/claude_code_docs_map.md help me configure hooks
```

Claude will fetch the map, find the right page, and answer from the actual docs instead of guessing.

### Planning vs. Thinking

Breadth vs. depth. Note that both of these increase token consumption.

Planning Mode is best for:

* Tasks requiring broad understanding of your codebase
* Multi-step implementations
* Changes that affect multiple files or components

Thinking Mode is best for:

* Complex logic problems
* Debugging difficult issues
* Algorithmic challenges

## Context Management

| Command | What it does |
|---|---|
| `/clear` | Reset context window (keeps memory); useful when switching to a completely different task and you want no previous context |
| `/compact` | Summarize and compress conversation to free context; useful after it learns something useful that you would like it to retain |
| `Esc` | Interrupt a running response |
| `Esc Esc` | Rewind conversation by a message; useful when you can drop unhelpful context (e.g., extensive debugging on a particularly niche issue) |

## Customizing Behavior

### Skills vs. CLAUDE.md vs. Slash Commands vs. Hook vs. Plugins

Claude Code has several ways to customize behavior. Skills are unique because they're automatic and task-specific. Here's how they compare:

* CLAUDE.md files load into every conversation. If you want Claude to always use TypeScript's strict mode, that goes in CLAUDE.md.
* Skills load on demand when they match your request. Claude only loads the name and description initially, so they don't fill up your entire context window. If you find yourself explaining the same thing to Claude repeatedly, that's a skill waiting to be written.
* Slash commands require you to explicitly type them. Skills don't. Claude applies them when it recognizes the situation.
* Hooks are shell commands that run automatically on specific events (e.g., before/after a tool call, when Claude stops). They execute outside of Claude's context — use them for side effects like logging, formatting, or notifications that should always happen regardless of what Claude does.
* Plugins extend Claude Code itself with new tools, MCP servers, or integrations.

## Thariq (Anthropic) posts on X

[Compilation](https://x.com/trq212/status/2035372716820218141?s=20) as of 2026-03-21.

### 2025-09-22 your agent should use a file system

Context windows will not get long enough to store entire codebases. And that's not the point -- the point is to give the agent or Claude Code the tools to find what it needs. It doesn't need everything.

### 2025-09-29 Claude Agent SDK

The agent SDK is the easiest way to build agents.

### 2025-10-27 bash is all you need

Even non-coding agents need bash.

This lets the model:  
- ground its results in reproducible code 
- take multiple steps at finding everything 
- double check its work and verify it

### 2025-12-28 Building large features

Spec-based. Start with a minimal spec or prompt and ask Claude to interview you using the `AskUserQuestionTool`.

```
read this @SPEC.md and interview me in detail using the AskUserQuestionTool about literally anything: technical implementation, UI & UX, concerns, tradeoffs, etc. but make sure the questions are not obvious

be very in-depth and continue interviewing me continually until it's complete, then write the spec to the file
```

### 2026-01-29 playgrounds are one of the best ways to iterate on ideas visually

[Examples](https://x.com/trq212/status/2017024445244924382?s=20) from Thariq.

### 2026-02-19 prompt caching

Critical, when you are building agents from scratch. Otherwise token cost easily can jump massively.

### 2026-02-27 building an agent is more of an art than a science

[Original post.](https://x.com/trq212/status/2027463795355095314)

* You want to give your agent tools that are shaped to its own abilities. But how do you know what those abilities are? You pay attention, read its outputs, experiment. You learn to see like an agent. **As model capabilities increase, the tools that your models once needed might now be constraining them.** It’s important to constantly revisit previous assumptions on what tools are needed.
* This is a pattern we’ve seen as Claude gets smarter, it becomes increasingly good at building its context if it’s given the right tools. 
* Progressive disclosure: instead of dumping everything into the prompt upfront, you give the agent just enough to know where to look, and let it pull in more detail only when it needs it. Add more functionality without adding new tools (or unnecessarily bloating the context window).
* Must clear a high bar to add another tool. Becomes another option that Claude Code has to consider whether to use or not.

### 2026-03-17 skills are the abstraction that all agents will build on