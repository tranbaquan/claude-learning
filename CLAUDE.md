# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a Python-based API testing practice repository focused on authentication and authorization workflows. The project emphasizes learning test-driven development patterns for REST APIs using pytest.


## Development Workflow

### Test Development Process
1. **Requirements Phase**: Read feature documents in `docs/` to understand API endpoints and expected behaviors
2. **Task Breakdown**: For complex features, use the `feature-task-generator` agent to decompose specs into actionable tasks
3. **Implementation**: Use the `feature-implementer` agent to write test code from task specifications
4. **Validation**: Run tests with `pytest -v` to verify implementation
