# AI Agent Guide

A practical guide for using Claude Code agents and skills to scaffold servers and generate tests.

## Table of Contents

- [Setup](#setup)
- [Server Setup with Coding Agent](#server-setup-with-coding-agent)
- [Test Generation with Coding Agent](#test-generation-with-coding-agent)

## Setup

| Topic               | Reference                                                |
|---------------------|----------------------------------------------------------|
| MCP configuration   | [claude-docs/setup/mcp.md](claude-docs/setup/mcp.md)     |
| Skill configuration | [claude-docs/setup/skill.md](claude-docs/setup/skill.md) |

## Server Setup with Coding Agent

Examine code generation capabilities by driving a `server-setup` skill from instruction documents.

### Prerequisites

- Server setup guides under `docs/server/`
- `server-setup` skill at `.claude/skills/server-setup/` that reads the instructions and implements them
- Playwright skill or MCP tools for UI verification —
  see [.claude/skills/playwright-cli/SKILL.md](.claude/skills/playwright-cli/SKILL.md)

### Usage

Run the `server-setup` skill against each instruction document:

```shell
claude --permission-mode bypassPermissions -p "/server-setup docs/server/01-setup-server.md"
claude --permission-mode bypassPermissions -p "/server-setup docs/server/02-auth.md"
claude --permission-mode bypassPermissions -p "/server-setup docs/server/03-register-page.md"
claude --permission-mode bypassPermissions -p "/server-setup docs/server/04-login-page.md"
```

> Add more instruction documents under `docs/server/` and re-run the command to scaffold additional servers or features.

## Test Generation with Coding Agent

Examine code generation capabilities by delegating to specialized testing agents.

### Prerequisites

- Testing docs under `docs/testing/`
- Two agents that read the instructions and implement tests:
    - `.claude/agents/api-testing.md` — generates API tests
    - `.claude/agents/ui-testing.md` — generates UI tests
- Playwright skill or MCP tools for UI verification —
  see [.claude/skills/playwright-cli/SKILL.md](.claude/skills/playwright-cli/SKILL.md)

### Usage

Run each agent against the same testing document to produce API and UI tests in parallel:

```shell
# API tests
claude --permission-mode bypassPermissions --agent api-testing -p "implement test for @docs/testing/01-test-auth.md"

# UI tests
claude --permission-mode bypassPermissions --agent ui-testing -p "implement test for @docs/testing/01-test-auth.md"
```

> Both commands share the same instruction but are routed to different agents, producing API and UI test suites
> independently. Swap in other documents under `docs/testing/` to cover additional features.
