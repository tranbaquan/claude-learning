# AI Agent Guide

## Setup Server With Coding Agent

### Pre-requisites

1. A server setup guide in `docs/server`.
2. A skill for server setup in `.claude/skills/server-setup`, which will read the instruction from the document
   and implement the server setup.

### Steps

Follow the command below to run the server setup skill with the instruction document:

```shell
claude --permission-mode bypassPermissions -p "/server-setup docs/server/01-setup-server.md"
claude --permission-mode bypassPermissions -p "/server-setup docs/server/02-auth.md"
```

You can add more instruction documents and run the same command to set up different servers or implement different
features.