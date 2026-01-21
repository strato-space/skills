---
name: close-session
description: "Close out a work session by accepting all repo changes, updating AGENTS.md/CHANGELOG.md/README.md, preparing an extended git commit, pushing, and sending a Telegram changelog via tgbot__send_bot_message using /home/strato-space/settings/routing-prod.json. Use when the user asks to Close Session, Закрыть сессию, Share, Зашарить, or to finalize a session with changelog + Telegram broadcast."
---

# Close Session

## Overview

Close a work session end-to-end: accept all changes, update documentation, curate the changelog, commit and push, then broadcast the changelog to Telegram using routing rules.

## Workflow

### 1) Collect repo context and diff

- Run `git status` and `git diff` to understand the change set.
- If `git diff` is empty, stop and report that there is nothing to close.
- Capture repo context for routing:
  - Repo root: `git rev-parse --show-toplevel`
  - Origin URL: `git remote get-url origin`
  - Subdir relative to root: `git rev-parse --show-prefix` (empty means root; treat as `.`)
- Accept all changes, including changes made outside the session. Do not discard or revert anything.

### 2) Update documentation (English)

- Update `AGENTS.md`, `CHANGELOG.md`, and `README.md` to reflect the work done.
- Keep documentation in English, including any updates in `docs/*`.
- Enforce changelog structure for each date section:
  - `## YYYY-MM-DD`
  - `### PROBLEM SOLVED` (business impact; symptom -> effect -> solution)
  - `### FEATURE IMPLEMENTED` (business value)
  - `### CHANGES` (technical changes: APIs/configs/files/migrations)
- If a section has no entries, include `- None.` to keep the structure consistent.

### 3) Prepare the extended commit message

- Use `git diff` and the updated docs to craft an English commit message with a subject and body.
- Keep it explicit and scoped to the actual changes.
- Template:

```text
<short summary>

Problem solved:
- ...

Feature implemented:
- ...

Changes:
- ...

Tests:
- Not run (not requested)
```

### 4) Commit and push

- Stage everything: `git add -A` (include untracked, modified, and deleted files).
- Commit with the extended message.
- Push the current branch.
- If needed for context, check GitHub state with `gh auth status` and `gh pr status`.

### 5) Prepare the Telegram changelog (Russian)

- Determine the changelog range to send:
  - If a previous Telegram send date is known (from an existing log or explicit record), include all dates since that send (inclusive or from the next date, depending on the record).
  - If no previous send is known, include only today's date.
- The Telegram message must be in Russian and include the full changelog content for the chosen range.
- Emphasize PROBLEM SOLVED and FEATURE IMPLEMENTED items and bold the most important points.
- Format as Telegram MarkdownV2:
  - Header: `*{topic}* - {YYYY\-MM\-DD}` (use the latest date included).
  - Break text into paragraphs.
- Escape MarkdownV2 special characters in dynamic text (topic, dates, changelog items) by prefixing with `\`.
  - Escape set:

```text
_ * [ ] ( ) ~ ` > # + - = | { } . !
```

### 6) Resolve routing and send via MCP

- Load `/home/strato-space/settings/routing-prod.json` (array of routing-item-template).
- Match by `sources[].github`:
  - `github.url` must equal the repo origin URL.
  - `github.path` must match the current subdir:
    - `.` matches only the repo root.
    - Otherwise, match when the current subdir equals `github.path` or starts with `github.path/`.
- For each matching routing item, iterate its `output` array and find `tgbot:send_bot_message`.
- Build the Telegram target:
  - `chat_id`: prefix `-100` to the provided chat id if it does not already start with `-100`.
  - `message_thread_id`: use `thread_id` when present.
- Deduplicate by `(chat_id, message_thread_id)` and send once per unique target.
- Send with `tgbot__send_bot_message` using `parse_mode=MARKDOWN` (MarkdownV2 style) and the prepared Russian message.
- If no routing items match, stop and report the missing routing configuration.
