# CLAUDE.md

## Project Overview
This is a Python-based API testing.

## Development Workflow

### Test Development Process
1. **Requirements Phase**: Read feature documents in `docs/` to understand API endpoints and expected behaviors
2. **Task Breakdown**: For complex features, use the `feature-task-generator` agent to decompose specs into actionable tasks
3. **Implementation**: Use the `feature-implementer` agent to write test code from task specifications
4. **Validation**: Run tests with `pytest -v` to verify implementation

## Coding Conventions
- Follow coding style guidelines in @.claude/rules/pytest-code-style.md
