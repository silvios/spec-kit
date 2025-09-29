---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
scripts:
  sh: ./.specify/scripts/bash/setup-plan.sh --json
  ps: ./.specify/scripts/powershell/setup-plan.ps1 -Json
---

The user input to you can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

The user input after `/<command>` can follow two formats:
1.  **Mono Repo:** `/<command> <project-name> <implementation-details>` (e.g., `/plan @my-app use .NET Aspire...`)
2.  **Single Project:** `/<command> <implementation-details>` (e.g., `/plan use .NET Aspire...`)

You **MUST** parse the arguments to determine the context. The `implementation-details` are for you to use later in this prompt; they are NOT passed to the script.

Given the arguments, do this:

1.  **Parse Arguments**:
    *   Check if the first argument starts with `@`.
    *   **If it does**:
        *   The first argument is the `PROJECT_NAME`.
        *   Construct the `PROJECT_PATH` like this: `./projects/PROJECT_NAME`.
        *   The rest of the arguments are the `IMPLEMENTATION_DETAILS`, which you will use later.
    *   **If it does not**:
        *   There is no `PROJECT_NAME` or `PROJECT_PATH`.
        *   All arguments are the `IMPLEMENTATION_DETAILS`, which you will use later.

2.  **Execute Script**:
    *   Run the script defined in `{SCRIPT}` from the repository root.
    *   If you identified a `PROJECT_PATH`, you **must** pass it to the script using the `--project-path` (for .sh) or `-ProjectPath` (for .ps1) argument.
    *   **Example (Mono Repo, sh):** `./.specify/scripts/bash/setup-plan.sh --json --project-path ./projects/@my-app`
    *   **Example (Single Project, sh):** `./.specify/scripts/bash/setup-plan.sh --json`
    *   Parse the script's JSON output for `FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, and `BRANCH`. All file paths must be absolute.
   - BEFORE proceeding, inspect FEATURE_SPEC for a `## Clarifications` section with at least one `Session` subheading. If missing or clearly ambiguous areas remain (vague adjectives, unresolved critical choices), PAUSE and instruct the user to run `/clarify` first to reduce rework. Only continue if: (a) Clarifications exist OR (b) an explicit user override is provided (e.g., "proceed without clarification"). Do not attempt to fabricate clarifications yourself.
2. Read and analyze the feature specification to understand:
   - The feature requirements and user stories
   - Functional and non-functional requirements
   - Success criteria and acceptance criteria
   - Any technical constraints or dependencies mentioned

3. Read the constitution at `./.specify/memory/constitution.md` to understand constitutional requirements.

4. Execute the implementation plan template:
   - Load `./.specify/templates/plan-template.md` (already copied to IMPL_PLAN path)
   - Set Input path to FEATURE_SPEC
   - Run the Execution Flow (main) function steps 1-9
   - The template is self-contained and executable
   - Follow error handling and gate checks as specified
   - Let the template guide artifact generation in $SPECS_DIR:
     * Phase 0 generates research.md
     * Phase 1 generates data-model.md, contracts/, quickstart.md
     * Phase 2 generates tasks.md
   - Incorporate user-provided details from arguments into Technical Context: {{args}}
   - Update Progress Tracking as you complete each phase

5. Verify execution completed:
   - Check Progress Tracking shows all phases complete
   - Ensure all required artifacts were generated
   - Confirm no ERROR states in execution

6. Report results with branch name, file paths, and generated artifacts.

Use absolute paths with the repository root for all file operations to avoid path issues.
