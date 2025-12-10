- Always use descriptive function and variable names even if they are verbose
- Always use rg over grep
- Always use fd over find
- Always prefer typescript over javascript
- Always use biome for typescript
- Always prefer pnpm over npm
- Always prefer uv over pip
- Always typecheck, lint and format after finishing a task and fix all errors, use the static-analysis agent if available
- Use "pnpm check" for typescript if available, if not "pnpm typecheck", "pnpm lint", and "pnpm format"
- Always use "cargo clippy --all-targets --all-features" for rust linting
- Always use "cargo fmt" for rust formatting
- Always use "uv run ruff check --fix" for python linting
- Always use "uv run ruff format" for python formatting
- Always use "uv run ty check" for python typechecking, not mypy or pyright

# Common setup for package.json

```
"lint": "biome check --write",
"typecheck": "tsc --noEmit",
"check": "pnpm run lint && pnpm run typecheck",
```

# Notifications

Use the telegram-notifier agent to send messages when:
  - A task is complete
  - You need user input to continue
  - An error occurs that requires user attention
  - The user explicitly asks for a notification (e.g., "notify me", "send me a message", "let me know")

# Large Codebase Analysis

When analyzing large codebases or multiple files that might exceed context limits, use the gemini-codebase-analyzer agent with its massive context window to leverage Google Gemini's large context capacity.
