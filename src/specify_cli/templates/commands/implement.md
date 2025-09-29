---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

The user input can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

The user input after `/<command>` can follow two formats:
1.  **Mono Repo:** `/<command> <project-name> [optional-args]` (e.g., `/implement @my-app`)
2.  **Single Project:** `/<command> [optional-args]` (e.g., `/implement`)

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
    *   **Example (Mono Repo, sh):** `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks --project-path ./projects/@my-app`
    *   **Example (Single Project, sh):** `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`
    *   Parse the script's JSON output for `FEATURE_DIR` and `AVAILABLE_DOCS`. All paths must be absolute.

2. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

3. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements

4. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together  
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

5. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

6. Progress tracking and error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **IMPORTANT** For completed tasks, make sure to mark the task off as [X] in the tasks file.

7. Completion validation:
   - Verify all required tasks are completed
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - Report final status with summary of completed work

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/tasks` first to regenerate the task list.