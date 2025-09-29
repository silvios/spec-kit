---
description: Create or update the feature specification from a natural language feature description.
scripts:
  sh: ./.specify/scripts/bash/create-new-feature.sh --json
  ps: ./.specify/scripts/powershell/create-new-feature.ps1 -Json
---

The user input to you can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

The user input after `/specify` can follow two formats:
1.  **Mono Repo:** `/<command> <project-name> <feature-description>` (e.g., `/specify @my-app build a login page`)
2.  **Single Project:** `/<command> <feature-description>` (e.g., `/specify build a login page`)

You **MUST** parse the arguments to determine the context.

Given the arguments, do this:

1.  **Parse Arguments**:
    *   Check if the first argument starts with `@`.
    *   **If it does**:
        *   The first argument is the `PROJECT_NAME`.
        *   Construct the `PROJECT_PATH` like this: `./projects/PROJECT_NAME`.
        *   The rest of the arguments are the `FEATURE_DESCRIPTION`.
    *   **If it does not**:
        *   There is no `PROJECT_NAME` or `PROJECT_PATH`.
        *   All arguments are the `FEATURE_DESCRIPTION`.

2.  **Execute Script**:
    *   Run the script defined in `{SCRIPT}` from the repository root.
    *   Append the `FEATURE_DESCRIPTION` as arguments to the script.
    *   If you identified a `PROJECT_PATH`, you **must** pass it to the script using the `--project-path` (for .sh) or `-ProjectPath` (for .ps1) argument.
    *   **Example (Mono Repo, sh):** `./.specify/scripts/bash/create-new-feature.sh --json --project-path ./projects/@my-app build a login page`
    *   **Example (Single Project, sh):** `./.specify/scripts/bash/create-new-feature.sh --json build a login page`
    *   Parse the script's JSON output for `BRANCH_NAME` and `SPEC_FILE`. All file paths must be absolute.

  **IMPORTANT**: You must only ever run this script once per command.
2. Load `./.specify/templates/spec-template.md` to understand required sections.
3. Write the specification to SPEC_FILE using the template structure, replacing placeholders with concrete details derived from the feature description (arguments) while preserving section order and headings.
4. Report completion with branch name, spec file path, and readiness for the next phase.

Note: The script creates and checks out the new branch and initializes the spec file before writing.
