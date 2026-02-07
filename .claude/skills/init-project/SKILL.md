---
name: init-project
description: Scaffold a new project with opinionated tooling configs for TypeScript, Rust, and/or Python
argument-hint: "[languages: ts|rust|python...] [project-name]"
---

# init-project

Scaffold a new project directory with opinionated tooling configurations, code quality checks, and documentation.

## Argument Parsing

Parse the arguments from the user's invocation. Arguments are space-separated and can appear in any order.

**Language tokens** (case-insensitive):
- `ts`, `typescript` → TypeScript
- `rust` → Rust
- `python`, `py` → Python

**Project name**: The remaining non-language token. If not provided, ask the user for a project name.

If no languages are specified, ask the user which languages to include.

## Execution Flow

### 1. Determine Project Layout

**Single language**: Everything goes at the project root.

**Multiple languages**: Ask the user two questions:
1. "Should each language live in its own subdirectory, or should everything be at the project root?"
2. If subdirectories: "What should each subdirectory be named?" (ask for each language — do NOT assume defaults like `app/` or `server/`)

### 2. Create Project Directory

Create the project directory at `./<project-name>/` relative to the current working directory.

Initialize git:
```bash
git init <project-name>
```

### 3. Generate Shared Files (always created at project root)

#### `.gitignore`

Generate a `.gitignore` with entries conditional on the selected languages:

**Always include:**
```
.DS_Store
.env
.env.*
!.env.example
```

**If TypeScript:**
```
node_modules/
dist/
.turbo/
```

**If Rust:**
```
target/
```

**If Python:**
```
__pycache__/
*.pyc
.venv/
*.egg-info/
```

#### `AGENTS.md`

Create with this exact content:
```markdown
Read [CONTRIBUTING.md](./CONTRIBUTING.md).
```

#### `CLAUDE.md`

Create as a symlink pointing to `AGENTS.md`:
```bash
ln -s AGENTS.md CLAUDE.md
```

#### `CONTRIBUTING.md`

Generate adapted to the included languages. Use this template:

```markdown
# Contributing

## Code Quality

{include only sections for selected languages}
```

**If TypeScript is included:**
```markdown
### TypeScript

```bash
pnpm lint       # Biome linting with auto-fix
pnpm typecheck  # TypeScript type checking
pnpm check      # Run all checks
```
```

**If Rust is included:**
```markdown
### Rust

```bash
cargo clippy --all-targets --all-features  # Linting
cargo fmt                                   # Formatting
```
```

**If Python is included:**
```markdown
### Python

```bash
uv run ruff check --fix  # Linting with auto-fix
uv run ruff format       # Code formatting
uv run ty check          # Type checking
```
```

**Always include this full section after Code Quality:**

```markdown
## Code Style & Philosophy

### Typing & Pattern Matching

- Prefer **explicit types** over raw dicts -- make invalid states unrepresentable where practical
- Prefer **typed variants over string literals** when the set of valid values is known
- Use **exhaustive pattern matching** (`match` in Python and Rust, `ts-pattern` in TypeScript) so the type checker can verify all cases are handled
- Structure types to enable exhaustive matching when handling variants
- Prefer **shared internal functions over factory patterns** when extracting common logic from hooks or functions -- keep each export explicitly defined for better IDE navigation and readability

### Forward Compatibility

- **Unknown values**: Parse to an explicit `Unknown*` variant (never `None`), log at warn level, preserve raw data, gracefully ignore instead of raising exception

### Self-Documenting Code

- **Verbose naming**: Variable and function naming should read like documentation
- **Strategic comments**: Only for non-obvious logic or architectural decisions; avoid restating what code shows
```

### 4. Generate Language-Specific Files

Place these files in the appropriate directory (root for single-language, or the user-chosen subdirectory for multi-language).

#### If TypeScript

**`package.json`:**
```json
{
  "name": "<project-or-subdirectory-name>",
  "version": "0.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "",
    "build": "",
    "lint": "biome check --write",
    "typecheck": "tsc --noEmit",
    "check": "pnpm run lint && pnpm run typecheck"
  },
  "devDependencies": {
    "@biomejs/biome": "latest",
    "typescript": "latest"
  }
}
```

If Rust is co-located in the same repo, ask the user: "Do you want cargo check commands (clippy, fmt) wired into the package.json scripts?"

If yes, add these scripts (adjust `--manifest-path` if Rust is in a subdirectory):
```json
"cargo:clippy": "cargo clippy --manifest-path <path>/Cargo.toml --all-targets --all-features",
"cargo:fmt": "cargo fmt --manifest-path <path>/Cargo.toml",
"cargo": "pnpm run cargo:clippy && pnpm run cargo:fmt",
"check": "pnpm run lint && pnpm run typecheck && pnpm run cargo"
```

**`tsconfig.json`:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["DOM", "DOM.Iterable", "ESNext"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "noEmit": true,
    "strict": true,
    "noImplicitReturns": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noPropertyAccessFromIndexSignature": true,
    "useUnknownInCatchVariables": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "incremental": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"]
}
```

**`biome.json`:**
```json
{
  "$schema": "./node_modules/@biomejs/biome/configuration_schema.json",
  "vcs": { "enabled": true, "clientKind": "git", "useIgnoreFile": true },
  "files": { "ignoreUnknown": true },
  "formatter": { "enabled": true, "indentStyle": "tab" },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedVariables": "error",
        "noUnusedImports": "error",
        "noUnusedFunctionParameters": "error"
      },
      "suspicious": { "noExplicitAny": "warn" }
    }
  },
  "javascript": { "formatter": { "quoteStyle": "double" } },
  "assist": { "enabled": true, "actions": { "source": { "organizeImports": "on" } } }
}
```

**`src/` directory**: Create an empty `src/` directory with a placeholder file:
```bash
mkdir -p src
```

#### If Rust

**`Cargo.toml`:**
```toml
[package]
name = "<project-or-subdirectory-name>"
version = "0.1.0"
edition = "2021"

[dependencies]

[lints.clippy]
pedantic = { level = "warn", priority = -1 }
needless_pass_by_value = "allow"
unnecessary_wraps = "allow"
missing_panics_doc = "allow"
```

**`src/main.rs`:**
```rust
fn main() {
    println!("Hello, world!");
}
```

#### If Python

**`pyproject.toml`:**
```toml
[project]
name = "<project-or-subdirectory-name>"
version = "0.1.0"
description = ""
requires-python = ">=3.13"

[dependency-groups]
dev = ["ruff>=0.15.0", "ty>=0.0.15"]

[build-system]
requires = ["uv_build>=0.10.0"]
build-backend = "uv_build"

[tool.ruff]
line-length = 100
target-version = "py313"

[tool.ruff.lint]
select = ["E", "W", "F", "I", "B", "C4", "UP", "ANN", "PTH", "RET", "SIM", "TID", "RUF"]
ignore = ["E501", "B008", "C901", "RET504", "ANN401"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.ty.rules]
division-by-zero = "error"
deprecated = "error"
unused-ignore-comment = "error"
possibly-unresolved-reference = "error"
```

### 5. Post-Setup

After creating all files:

1. If TypeScript was included, run `pnpm install` in the TypeScript directory to install devDependencies
2. If Python was included, run `uv sync` in the Python directory
3. Report what was created with a summary of the project structure

Do NOT make an initial git commit — leave that to the user.
