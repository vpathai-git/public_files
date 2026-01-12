# VPath: VS Code MCP Operational Instructions (for Language Models)

This document is intended to be read by a language model operating inside VS Code with Model Context Protocol (MCP) tools enabled.

## Goals

- Make correct, minimal, verifiable edits to files in the workspace.
- Prefer tool-backed evidence over assumptions.
- Keep changes safe, incremental, and easy to review.

## Operating Principles

1. **Don't guess**
   - If you don't know a file path, search for it.
   - If you don't know a symbol definition, locate it.
   - If you don't know expected behavior, read existing tests/docs.

2. **Prefer read-only discovery before edits**
   - Start by listing directories and reading relevant files.
   - Search for usages before changing public APIs.

3. **Small, surgical changes**
   - Avoid unrelated refactors.
   - Preserve coding style and existing conventions.
   - Avoid reformatting unless required.

4. **Validate your work**
   - Check diagnostics after edits.
   - Use syntax validation before committing changes.

5. **Be explicit about files and operations**
   - Always name the file(s) you will edit.
   - Use the appropriate edit tool for modifications.

## Available Tools

You have access to the following tools that interact with VS Code and the workspace. Use them instead of "imagining" project state.

### Memory & Information

| Tool | Description |
|------|-------------|
| `save_memory` | Saves specific facts or preferences about the user for future interactions. |
| `google_web_search` | Performs a web search using Google to find information. |

### Task Management

| Tool | Description |
|------|-------------|
| `write_todos` | Manages a list of subtasks for complex requests. Use this to track progress on multi-step operations. |

### File System & Navigation

| Tool | Description |
|------|-------------|
| `vscode_list_directory` | Lists the contents of a directory. Use to explore folder structure and confirm file names. |
| `vscode_read_file` | Reads the content of a file. **Important:** This shows unsaved changes in the editor buffer. |
| `vscode_create_file` | Creates a new file with the specified content. |
| `vscode_delete_file` | Deletes a file or directory. |
| `vscode_move_file` | Moves or renames a file or directory. |
| `vscode_copy_file` | Copies a file or directory to a new location. |
| `vscode_get_file_info` | Gets information about a file or directory (stat: size, modified time, type). |
| `vscode_get_cwd` | Gets the current working directory of the workspace. |
| `vscode_find_files` | Finds files in the workspace using glob patterns. Examples: `**/*.ts`, `src/**/*.sysml` |

### Code Analysis & Navigation

| Tool | Description |
|------|-------------|
| `vscode_get_definition` | Go to Definition. Navigates to where a symbol is defined. |
| `vscode_get_references` | Find All References. Locates all usages of a symbol across the workspace. |
| `vscode_get_hover` | Gets hover information (types, documentation) at a specific position in a file. |
| `vscode_get_symbols` | Gets all symbols (functions, classes, variables) in a file. |
| `vscode_find_workspace_symbols` | Searches for symbols across the entire workspace by name. |
| `vscode_get_diagnostics` | Gets all errors and warnings for files in the workspace. |
| `vscode_diagnostics_for_file` | Gets diagnostics (errors, warnings) for a specific file. |
| `vscode_validate_syntax` | Checks a file for syntax errors before making changes. |

### Code Editing

| Tool | Description |
|------|-------------|
| `copilot_replaceString` | **Preferred tool for making edits.** Replaces exact text in a file. Provide enough context for unique matching. |
| `vscode_save_file` | Saves a file to disk. Use after making edits to persist changes. |
| `vscode_edit_insert` | Inserts text at a specific line and character position. |
| `vscode_edit_replace` | Replaces text in a specific range (start line/char to end line/char). |
| `vscode_edit_delete` | Deletes text in a specific range. |
| `vscode_apply_edit` | Applies atomic workspace edits across multiple files. |
| `vscode_rename_symbol` | Renames a symbol across the workspace using semantic understanding (refactoring rename). |

### Refactoring & Formatting

| Tool | Description |
|------|-------------|
| `vscode_refactor_get_actions` | Gets available refactoring actions at a position (extract method, inline, etc.). |
| `vscode_refactor_format_document` | Formats an entire document according to the configured formatter. |
| `vscode_refactor_organize_imports` | Organizes and sorts imports in a file. |

### Editor Control

| Tool | Description |
|------|-------------|
| `vscode_run_command` | Runs any VS Code command by its command ID. Useful for triggering built-in functionality. |

## Recommended Workflow (Checklist)

### 1. Clarify the target
- What file(s) are involved?
- What is the expected outcome?

### 2. Explore
- `vscode_list_directory` to see folder structure
- `vscode_find_files` to locate files by pattern
- `vscode_read_file` to examine content
- `vscode_get_symbols` to understand file structure

### 3. Assess impact
- `vscode_get_references` before changing public APIs
- `vscode_get_definition` to understand symbol origins
- Identify related tests or documentation

### 4. Edit
- Use `copilot_replaceString` for modifications (preferred)
- Include enough surrounding context for unique matching
- Prefer minimal, surgical changes
- Use `vscode_save_file` to persist changes

### 5. Validate
- `vscode_diagnostics_for_file` to check for errors
- `vscode_validate_syntax` to verify correctness
- Review the changes made

### 6. Summarize
- What changed (files + intent)?
- How to verify?
- Any known limitations?

## Tool Usage Best Practices

### Reading Files
```
vscode_read_file:
  - Reads the editor buffer (includes unsaved changes)
  - Prefer reading entire files when practical
  - Use vscode_get_file_info first if you need metadata
```

### Making Edits
```
copilot_replaceString (preferred):
  - Provide exact text to match
  - Include 3-5 lines of surrounding context for unique identification
  - Avoid matching text that appears multiple times without enough context

vscode_edit_* tools:
  - Use when you need precise line/character positioning
  - vscode_edit_insert for adding new content
  - vscode_edit_replace for modifying existing content
  - vscode_edit_delete for removing content
```

### Finding Information
```
vscode_find_files:
  - Use glob patterns: **/*.ts, src/**/*.sysml, **/test*.js
  - Returns matching file paths

vscode_find_workspace_symbols:
  - Searches symbol names across all files
  - Useful for finding class/function definitions by name

vscode_get_references:
  - Requires a file path and position
  - Returns all locations where the symbol is used
```

### Diagnostics
```
vscode_get_diagnostics:
  - Gets all problems in the workspace
  - Use after making changes to verify no errors introduced

vscode_diagnostics_for_file:
  - Focused check on a single file
  - More efficient when you know which file to check
```

## Safety and Quality Rules

- Never fabricate tool outputs.
- Never claim a command succeeded without actually running it.
- Don't edit generated artifacts unless the task explicitly requires it.
- Don't leak secrets: avoid printing tokens, keys, or private URLs.
- Always validate syntax after edits.
- Save files explicitly when changes should persist.

## SysML v2 Specific Guidance

When working with SysML v2 files (`.sysml`, `.kerml`):

1. **Use the language server features**
   - `vscode_get_diagnostics` will show SysML-specific errors
   - `vscode_get_definition` works for SysML symbols
   - `vscode_get_hover` provides type information

2. **Follow SysML v2 conventions**
   - Package structures should be properly nested
   - Use qualified names when referencing external elements
   - Maintain consistency with existing model patterns

3. **Validate before and after changes**
   - Check diagnostics before editing to understand baseline
   - Re-check after edits to ensure no new errors

## Extending the Instruction Registry

The file `sysml_v2_agent_instructions_registry.yaml` lists instruction documents.
To add more documents:

- Add another entry under `files:` with `id`, `title`, `description`, and `raw_url`.
- Prefer stable, versioned URLs if you need immutability (tagged releases).
