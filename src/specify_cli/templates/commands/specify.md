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

2.  **Sanitize Arguments**:
    *   Take the `FEATURE_DESCRIPTION` you just parsed.
    *   Create a new variable called `SANITIZED_FEATURE_DESCRIPTION`.
    *   In this new variable, replace every newline character (`\n`) with a single space.
    *   Trim any leading or trailing whitespace from the result.
    *   This will convert the potentially multi-line, formatted input into a single, clean string suitable for a command line.

3.  **Execute Script**:
    *   Run the script defined in `{SCRIPT}` from the repository root.
    *   Append the `SANITIZED_FEATURE_DESCRIPTION` as arguments to the script.
    *   If you identified a `PROJECT_PATH`, you **must** pass it to the script using the `--project-path` (for .sh) or `-ProjectPath` (for .ps1) argument.
    *   **Example (Mono Repo, sh):** `./.specify/scripts/bash/create-new-feature.sh --json --project-path ./projects/@my-app I am building a modern website for monitoring price changes...`
    *   **Example (Single Project, sh):** `./.specify/scripts/bash/create-new-feature.sh --json I am building a modern website for monitoring price changes...`
    *   Parse the script's JSON output for `BRANCH_NAME` and `SPEC_FILE`. All file paths must be absolute.

  **IMPORTANT**: You must only ever run this script once per command.
4. Load `./.specify/templates/spec-template.md` to understand required sections.
5. Write the specification to SPEC_FILE using the template structure, replacing placeholders with concrete details derived from the feature description (arguments) while preserving section order and headings.
6. Report completion with branch name, spec file path, and readiness for the next phase.

Note: The script creates and checks out the new branch and initializes the spec file before writing.
