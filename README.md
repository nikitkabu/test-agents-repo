# php-service-agents

Claude Code subagents for working on a Symfony PHP service.

## What's inside

```
.claude/
└── agents/
    ├── symfony-coder.md    — writes/refactors Symfony code (controllers, services, entities, migrations)
    ├── code-reviewer.md    — reviews diffs for correctness, typing, security, Symfony idioms
    └── test-writer.md      — writes PHPUnit / Pest tests, detects the project's conventions
CLAUDE.md                   — project-level orchestration guidance for the main agent
```

Each subagent is a markdown file with YAML frontmatter (`name`, `description`, `tools`) and a system prompt. Claude Code loads every `.md` file under `.claude/agents/` as an available subagent and delegates to one automatically based on its `description`, or on explicit invocation (`Use the test-writer subagent to cover the new OrderService`).

## How the agents work together

```
     user request
          │
          ▼
   ┌──────────────┐
   │ main agent   │  ← reads CLAUDE.md, plans, dispatches
   └──────┬───────┘
          │
   ┌──────┼─────────────────┐
   ▼      ▼                 ▼
 symfony-coder  →  test-writer  →  code-reviewer
   writes code     writes tests      reviews diff
```

The main agent will not finalize a task until `code-reviewer` returns without blocking issues.

## Using in a Symfony project

Pick whichever fits your workflow.

### Option A — copy into an existing project

```bash
git clone git@github.com:<your-user>/php-service-agents.git /tmp/agents
cp -r /tmp/agents/.claude /path/to/your/symfony-project/
cp /tmp/agents/CLAUDE.md /path/to/your/symfony-project/
cd /path/to/your/symfony-project && git add .claude CLAUDE.md && git commit -m "Add Claude subagents"
```

### Option B — git subtree (keep updates flowing)

```bash
# One-time setup inside your Symfony project
git subtree add --prefix=.claude \
  git@github.com:<your-user>/php-service-agents.git main --squash

# Later, pull updates
git subtree pull --prefix=.claude \
  git@github.com:<your-user>/php-service-agents.git main --squash
```

### Option C — git submodule

```bash
git submodule add git@github.com:<your-user>/php-service-agents.git .claude-agents
ln -s .claude-agents/.claude .claude
ln -s .claude-agents/CLAUDE.md CLAUDE.md
```

## Invoking the agents

Inside Claude Code, after the `.claude/agents/` directory is present:

- **Automatic** — just ask naturally. "Add an endpoint `POST /api/orders` that …" — the main agent will route to `symfony-coder`, then `test-writer`, then `code-reviewer`.
- **Explicit** — `Use the code-reviewer subagent to review the last commit.` / `Ask test-writer to cover OrderCancellationHandler.`

To list available subagents inside Claude Code, run `/agents`.

## Extending

Add a new subagent by dropping a markdown file into `.claude/agents/`. Template:

```markdown
---
name: my-agent
description: <one short sentence. Start with "Use PROACTIVELY when …" for auto-delegation.>
tools: Read, Write, Edit, Bash, Grep, Glob
---

You are …

## Rules
…

## Reporting format
…
```

Keep the `description` sharp — the main agent routes on it. Keep the system prompt opinionated — vague prompts produce vague work.

## Adapting to your codebase

The three agents ship with generic Symfony conventions. If your service has project-specific rules (e.g. CQRS with separate command/query buses, DDD layering, custom lint config), edit the corresponding agent's system prompt — those are the exact rules the subagent will follow on every task.
