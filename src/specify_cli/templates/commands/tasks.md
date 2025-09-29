---
description: Generate an actionable, dependency-ordered tasks.md for the feature based on available design artifacts.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json
  ps: scripts/powershell/check-prerequisites.ps1 -Json
---

The user input to you can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

The user input after `/<command>` can follow two formats:
1.  **Mono Repo:** `/<command> <project-name> [optional-args]` (e.g., `/tasks @my-app`)
2.  **Single Project:** `/<command> [optional-args]` (e.g., `/tasks`)

You **MUST** parse the arguments to determine the context.

Given the arguments, do this:

1.  **Parse Arguments**:
    *   Check if the first argument starts with `@`.
    *   **If it does**:
        *   The first argument is the `PROJECT_NAME`.
        *   Construct the `PROJECT_PATH` like this: `./projects/PROJECT_NAME`.
    *   **If it does not**:
        *   There is no `PROJECT_NAME` or `PROJECT_PATH`.

2.  **Execute Script**:
    *   Run the script defined in `{SCRIPT}` from the repository root.
    *   If you identified a `PROJECT_PATH`, you **must** pass it to the script using the `--project-path` (for .sh) or `-ProjectPath` (for .ps1) argument.
    *   **Example (Mono Repo, sh):** `scripts/bash/check-prerequisites.sh --json --project-path ./projects/@my-app`
    *   **Example (Single Project, sh):** `scripts/bash/check-prerequisites.sh --json`
    *   Parse the script's JSON output for `FEATURE_DIR` and `AVAILABLE_DOCS`. All paths must be absolute.
2. Load and analyze available design documents:
   - Always read plan.md for tech stack and libraries
   - IF EXISTS: Read data-model.md for entities
   - IF EXISTS: Read contracts/ for API endpoints
   - IF EXISTS: Read research.md for technical decisions
   - IF EXISTS: Read quickstart.md for test scenarios

   Note: Not all projects have all documents. For example:
   - CLI tools might not have contracts/
   - Simple libraries might not need data-model.md
   - Generate tasks based on what's available

3. Generate tasks following the template:
   - Use `/templates/tasks-template.md` as the base
   - Replace example tasks with actual tasks based on:
     * **Setup tasks**: Project init, dependencies, linting
     * **Test tasks [P]**: One per contract, one per integration scenario
     * **Core tasks**: One per entity, service, CLI command, endpoint
     * **Integration tasks**: DB connections, middleware, logging
     * **Polish tasks [P]**: Unit tests, performance, docs

4. Task generation rules:
   - Each contract file → contract test task marked [P]
   - Each entity in data-model → model creation task marked [P]
   - Each endpoint → implementation task (not parallel if shared files)
   - Each user story → integration test marked [P]
   - Different files = can be parallel [P]
   - Same file = sequential (no [P])

5. Order tasks by dependencies:
   - Setup before everything
   - Tests before implementation (TDD)
   - Models before services
   - Services before endpoints
   - Core before integration
   - Everything before polish

6. Include parallel execution examples:
   - Group [P] tasks that can run together
   - Show actual Task agent commands

7. Create FEATURE_DIR/tasks.md with:
   - Correct feature name from implementation plan
   - Numbered tasks (T001, T002, etc.)
   - Clear file paths for each task
   - Dependency notes
   - Parallel execution guidance

Context for task generation: {ARGS}

The tasks.md should be immediately executable - each task must be specific enough that an LLM can complete it without additional context.
