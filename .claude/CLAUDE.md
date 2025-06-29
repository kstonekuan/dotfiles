- Always prefer typescript over javascript
- Always use biome for typescript
- Always prefer pnpm over npm
- Always use descriptive function and variable names even if they are verbose
- Always build, typecheck and lint after finishing a task and fix all errors
- Always use clippy for rust
- Always prefer uv over pip
- Always use ruff for python
- Always prefer rg over grep
- Always prefer fd over find

# Telegram Notifications

Use the mcp__telegram-notify__send_telegram_message and mcp__discord-notify__send_discord_message tools to send notifications to Telegram and Discord at the same time when they are available.

- Always send a Telegram notification when:
  - A task is fully complete
  - You need user input to continue
  - An error occurs that requires user attention
  - The user explicitly asks for a notification (e.g., "notify me", "send me a message", "let me know")

- Include relevant details in notifications:
  - For builds/tests: success/failure status and counts
  - For errors: the specific error message and file location

- Use concise, informative messages like:
  - "✅ Build completed successfully (2m 34s)"
  - "❌ Tests failed: 3/52 failing in auth.test.ts"
  - "⚠️ Need permission to modify /etc/hosts"
