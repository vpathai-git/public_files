# VPath: VS Code MCP Operational Instructions (for Language Models)

This document is intended to be read by a language model operating inside VS Code with Model Context Protocol (MCP) tools enabled.

## Goals

- Make correct, minimal, verifiable edits to files in the workspace.
- Prefer tool-backed evidence over assumptions.
- Keep changes safe, incremental, and easy to review.

## Operating Principles

1. **Don’t guess**
   - If you don’t know a file path, search for it.
   - If you don’t know a symbol definition, locate it.
   - If you don’t know expected behavior, read existing tests/docs.

2. **Prefer read-only discovery before edits**
   - Start by listing directories and reading relevant files.
   - Search for usages before changing public APIs.

3. **Small, surgical changes**
   - Avoid unrelated refactors.
   - Preserve coding style and existing conventions.
   - Avoid reformatting unless required.

4. **Validate your work**
   - Run the narrowest relevant tests/commands.
   - Re-check diagnostics after edits.

5. **Be explicit about files and operations**
   - Always name the file(s) you will edit.
   - Use a patch-based edit tool for modifications.

## Tooling Overview (MCP in VS Code)

You have access to tools that interact with the workspace.
Use them instead of “imagining” project state.

### Workspace navigation

- **List directory contents**: use when you need to see what files exist.
  - Tool: `list_dir`
  - Best for: exploring folder structure, confirming file names.

- **Search for files by glob**: use when you know a pattern.
  - Tool: `file_search`
  - Examples: `src/**`, `**/*.md`, `**/*test*.ts`

- **Search within files**: use when you’re hunting for symbols/strings.
  - Tool: `grep_search`
  - Prefer regex alternation (e.g., `parse|Parser|parsing`).

- **Semantic search**: use when you’re not sure what exact text appears.
  - Tool: `semantic_search`

### Reading code

- **Read file content by line range**:
  - Tool: `read_file`
  - Guidance:
    - Prefer fewer, larger reads (e.g., 1–200) over many tiny reads.
    - Re-read with an expanded range if needed.

### Editing code

- **Modify existing files**:
  - Tool: `apply_patch`
  - Guidance:
    - Use the smallest diff that accomplishes the task.
    - Include enough surrounding context to uniquely locate the change.
    - Avoid gratuitous formatting changes.

- **Create new files**:
  - Tool: `create_file`

- **Create directories**:
  - Tool: `create_directory`

### Diagnostics and correctness

- **Get VS Code problems/diagnostics**:
  - Tool: `get_errors`
  - Use after changes to ensure you didn’t introduce new errors.

### Terminals, builds, and tests

- **Run commands in a persistent terminal**:
  - Tool: `run_in_terminal`
  - Guidance:
    - Prefer direct commands (avoid nested shells) unless necessary.
    - Use absolute paths when changing directories.
    - Keep output manageable (use `head`, `tail`, `grep`).

- **If tasks.json is needed**:
  - Tool: `create_and_run_task`

### Notebook workflows

If a Jupyter notebook exists in the workspace:

- Get cell IDs: `copilot_getNotebookSummary`
- Run a code cell: `run_notebook_cell`
- Read existing output: `read_notebook_cell_output`
- Edit cells: `edit_notebook_file`

### Python + Pylance tools (when relevant)

If you are working on Python code:

1. **Configure environment** first
   - Tool: `configure_python_environment`

2. Run snippets safely (preferred over shell `python -c`):
   - Tool: `mcp_pylance_mcp_s_pylanceRunCodeSnippet`

3. Check syntax before running:
   - Tool: `mcp_pylance_mcp_s_pylanceSyntaxErrors`

4. Workspace import/analysis help:
   - `mcp_pylance_mcp_s_pylanceImports`
   - `mcp_pylance_mcp_s_pylanceInstalledTopLevelModules`
   - `mcp_pylance_mcp_s_pylanceSettings`

5. Safe refactors (when appropriate):
   - Tool: `mcp_pylance_mcp_s_pylanceInvokeRefactoring`

## Recommended Workflow (Checklist)

1. **Clarify the target**
   - What file(s) are involved?
   - What is the expected outcome?

2. **Explore**
   - `list_dir` / `file_search`
   - `grep_search` / `semantic_search`
   - `read_file`

3. **Assess impact**
   - Find usages before changing signatures.
   - Identify tests/validation commands.

4. **Edit**
   - Use `apply_patch` for modifications.
   - Prefer minimal diffs.

5. **Validate**
   - `get_errors`
   - `run_in_terminal` to run targeted tests or build steps.

6. **Summarize**
   - What changed (files + intent)?
   - How to verify?
   - Any known limitations?

## Editor and Language Server Operations

When the task involves “what the editor sees” or language-server behavior:

- Use `get_errors` to confirm diagnostics.
- Use `grep_search`/`semantic_search` to locate definitions/usages rather than relying on intuition.
- Keep changes aligned with TypeScript/Node/Python build tooling present in the repo.

## Safety and Quality Rules

- Never fabricate tool outputs.
- Never claim a command succeeded without actually running it.
- Don’t edit generated artifacts unless the task explicitly requires it.
- Don’t leak secrets: avoid printing tokens, keys, or private URLs.

## Extending the Instruction Registry

The file `instruction_files.yaml` is meant to list instruction documents.
To add more documents:

- Add another entry under `files:` with `id`, `title`, `description`, and `raw_url`.
- Prefer stable, versioned URLs if you need immutability (tagged releases).

