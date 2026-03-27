---
name: configuring-editor-settings
version: 1.0.0
user-invocable: false
description: >-
  Configure shared editor and IDE settings including .editorconfig for cross-editor
  formatting basics, VS Code workspace settings (.vscode/settings.json,
  extensions.json, launch.json), JetBrains shared configurations, and conflict
  resolution between editor configs and formatters. Use when standardizing editor
  behavior across a team, reviewing IDE configuration files, or resolving conflicts
  between .editorconfig and Prettier/ESLint.
  Triggered by: editorconfig, vscode settings, extensions.json, launch.json, editor
  configuration, IDE settings, workspace settings, format on save, recommended
  extensions, debug configuration, code workspace.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Editor Settings Configuration

Standards for shared editor and IDE configuration that ensure consistent formatting, linting, and debugging across all team members.

## Purpose

Inconsistent editor settings cause three problems: noisy diffs (tabs vs spaces, trailing whitespace), missed linting (developer doesn't have the right extension), and onboarding friction (new developer spends hours configuring their editor). Shared editor configuration solves all three by checking editor settings into version control as part of the project.

Use this skill when creating `.editorconfig`, VS Code workspace settings, recommended extensions lists, or debug configurations for a project.

---

## .editorconfig

The universal baseline — supported natively by most editors or via a plugin.

### Standard Configuration

```ini
# .editorconfig
# https://editorconfig.org

root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.{py,rs,go,java}]
indent_size = 4

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab

[*.{yml,yaml}]
indent_size = 2

[*.{json,jsonc}]
indent_size = 2
```

### Rules

| Setting | Value | Rationale |
|---|---|---|
| `root = true` | Always at the top | Stops editor from searching parent directories |
| `indent_style = space` | Default for most languages | Spaces render consistently everywhere |
| `end_of_line = lf` | Unix line endings | Prevents mixed line ending issues on Windows |
| `charset = utf-8` | Standard encoding | Prevents encoding-related display issues |
| `trim_trailing_whitespace = true` | Clean diffs | Eliminates whitespace-only changes |
| `insert_final_newline = true` | POSIX compliance | Many tools expect files to end with newline |
| `trim_trailing_whitespace = false` for Markdown | Preserve trailing spaces | Two trailing spaces = `<br>` in Markdown |
| `indent_style = tab` for Makefile | Required by Make | Makefiles require tab indentation for recipes |

### Language-Specific Indent Sizes

| Language | Indent Size | Convention Source |
|---|---|---|
| JavaScript / TypeScript | 2 | Community standard, Prettier default |
| HTML / CSS / SCSS | 2 | Community standard |
| JSON / YAML | 2 | Human readability |
| Python | 4 | PEP 8 |
| Rust | 4 | rustfmt default |
| Go | tabs | gofmt default (override with `indent_style = tab`) |
| Java / Kotlin | 4 | Google/Oracle style guides |

---

## VS Code Workspace Settings

### .vscode/settings.json

```jsonc
// .vscode/settings.json
// Team-shared workspace settings. Personal overrides go in User Settings.
{
  // Formatting
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit",
    "source.organizeImports": "explicit"
  },

  // Language-specific formatters
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  },
  "[sql]": {
    "editor.defaultFormatter": "dorzey.vscode-sqlfluff"
  },

  // File associations
  "files.associations": {
    "*.env.example": "dotenv",
    "Taskfile*.yml": "yaml"
  },

  // Search exclusions (performance)
  "search.exclude": {
    "**/node_modules": true,
    "**/.git": true,
    "**/dist": true,
    "**/__pycache__": true,
    "**/.pytest_cache": true
  },

  // File nesting (cleaner explorer)
  "explorer.fileNesting.enabled": true,
  "explorer.fileNesting.patterns": {
    "package.json": "package-lock.json, pnpm-lock.yaml, yarn.lock, .npmrc",
    "tsconfig.json": "tsconfig.*.json",
    ".env.example": ".env, .env.local, .env.test, .envrc"
  }
}
```

### What Belongs in Workspace Settings

| Include | Exclude |
|---|---|
| `editor.formatOnSave` | Theme and color settings |
| `editor.defaultFormatter` per language | Font family and size |
| `editor.codeActionsOnSave` | Keybindings |
| `files.associations` | Window/panel layout |
| `search.exclude` | Vim/Emacs mode settings |
| `explorer.fileNesting.patterns` | Personal productivity extensions |
| Language-specific formatter config | Telemetry settings |

**Rule:** Workspace settings cover project correctness (formatting, linting, file associations). Personal comfort (theme, font, keybindings) stays in User Settings.

---

## VS Code Extensions

### .vscode/extensions.json

```jsonc
// .vscode/extensions.json
{
  "recommendations": [
    // Formatting & Linting
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "charliermarsh.ruff",

    // Language Support
    "ms-python.python",
    "ms-python.vscode-pylance",

    // Database
    "mtxr.sqltools",
    "mtxr.sqltools-driver-pg",

    // DevOps
    "ms-azuretools.vscode-docker",
    "ms-vscode-remote.remote-containers",

    // Git
    "eamodio.gitlens"
  ],
  "unwantedRecommendations": [
    // Conflicts with project formatter
    "HookyQR.beautify"
  ]
}
```

### Extension Rules

| Rule | Rationale |
|---|---|
| Only include extensions the project needs | Don't force personal tools on the team |
| Include the formatter/linter extensions | Ensures `formatOnSave` works for everyone |
| Include language support extensions | Consistent IntelliSense and diagnostics |
| Use `unwantedRecommendations` for conflicting extensions | Prevents formatter fights |
| Don't include themes, icon packs, or productivity tools | These are personal preferences |

---

## VS Code Debug Configurations

### .vscode/launch.json

```jsonc
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: API Server",
      "type": "debugpy",
      "request": "launch",
      "module": "uvicorn",
      "args": ["app.main:app", "--reload", "--port", "8000"],
      "envFile": "${workspaceFolder}/.env",
      "cwd": "${workspaceFolder}"
    },
    {
      "name": "Python: Current File",
      "type": "debugpy",
      "request": "launch",
      "program": "${file}",
      "envFile": "${workspaceFolder}/.env",
      "cwd": "${workspaceFolder}"
    },
    {
      "name": "Python: pytest",
      "type": "debugpy",
      "request": "launch",
      "module": "pytest",
      "args": ["${file}", "-v"],
      "envFile": "${workspaceFolder}/.env",
      "cwd": "${workspaceFolder}"
    },
    {
      "name": "Node: Dev Server",
      "type": "node",
      "request": "launch",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "envFile": "${workspaceFolder}/.env",
      "cwd": "${workspaceFolder}"
    }
  ]
}
```

### Debug Configuration Rules

| Rule | Rationale |
|---|---|
| Use `${workspaceFolder}` variables | Portable across machines |
| Reference `.env` file via `envFile` | Consistent with project env var management |
| Include configurations for common workflows | Server, current file, test runner |
| Don't hardcode absolute paths | Breaks on other developers' machines |
| Name configurations descriptively | `Python: API Server` not `Launch` |

---

## Conflict Resolution

### .editorconfig vs Prettier

Both can set indent size and style. Resolution:

1. **Prettier reads `.editorconfig`** — it uses EditorConfig values as defaults
2. **Prettier config takes precedence** — if both specify `tabWidth`, Prettier config wins
3. **Best practice:** Let `.editorconfig` set the universal defaults and Prettier handle JavaScript/TypeScript-specific formatting

### .editorconfig vs ESLint

ESLint can enforce indentation rules. Resolution:

1. **Disable ESLint indent rule** when using Prettier — `eslint-config-prettier` handles this
2. **Let `.editorconfig` handle indentation** for non-JS files
3. **Let Prettier handle indentation** for JS/TS files

### VS Code settings vs .editorconfig

When both are present:

1. `.editorconfig` takes precedence for the settings it covers (indent, whitespace, newlines)
2. VS Code settings handle everything else (formatOnSave, codeActions, file associations)
3. They complement each other — `.editorconfig` for universal basics, VS Code for IDE-specific behavior

---

## Anti-Patterns

### 1. User-Specific Paths in Workspace Settings

```jsonc
{
  "python.defaultInterpreterPath": "/home/alice/.pyenv/versions/3.12.7/bin/python"
}
```

Breaks for every other developer. Use relative paths or let extensions auto-detect.

### 2. Overriding Personal Preferences

```jsonc
{
  "workbench.colorTheme": "One Dark Pro",
  "editor.fontSize": 14,
  "editor.fontFamily": "Fira Code"
}
```

These are personal choices. Workspace settings should only enforce project correctness.

### 3. Missing .editorconfig with Mixed Editors

Team uses VS Code, WebStorm, and Neovim but has no `.editorconfig`. Each editor uses its own defaults, creating whitespace diffs on every commit.

### 4. Recommending Conflicting Formatters

Including both `esbenp.prettier-vscode` and `HookyQR.beautify` in extensions.json. Developers get different formatting depending on which extension "wins" format-on-save.

### 5. No Debug Configurations

Every developer creates their own `launch.json` from scratch. Commit a shared `launch.json` with the most common debug scenarios.

---

## Checklist

Before committing editor configuration:

- [ ] `.editorconfig` present with `root = true`
- [ ] Indent style and size set for all file types used in project
- [ ] `trim_trailing_whitespace = false` for Markdown
- [ ] `indent_style = tab` for Makefile (if using Make)
- [ ] `.vscode/settings.json` has `formatOnSave` and language-specific formatters
- [ ] `.vscode/extensions.json` lists required extensions
- [ ] `.vscode/launch.json` has common debug configurations
- [ ] No user-specific paths in workspace settings
- [ ] No personal preference settings in workspace settings
- [ ] Formatter conflicts resolved (Prettier + EditorConfig + ESLint)

---

## Related Skills and Agents

- **`configuring-devcontainers` skill**: Devcontainer `customizations.vscode` section overlaps with workspace settings — this skill covers the non-container case
- **`authoring-task-runners` skill**: VS Code tasks.json can integrate with task runners — coordinate task definitions
- **`auditing-environment-completeness` skill**: Checks for presence of editor configuration files
- **`environment_scaffolder` agent**: Generates editor configuration as part of complete environment setup
- **`environment_auditor` agent**: Detects missing or inconsistent editor configuration
