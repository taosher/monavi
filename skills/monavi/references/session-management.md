# Session and task management

A monavi task is one agent session with its own tab group. Use a stable session name and reuse it for every command in that task.

## Name the session

```bash
monavi browser --session invoice-check open https://example.com
monavi browser --session invoice-check snapshot -i
monavi browser --session invoice-check close
```

If `MONAVI_TASK_ID` is set, use it as the task identity. Keep it stable for the whole session and start a new ID for a new session.

Monavi also derives a session from the working directory when you do not pass `--session`, but an explicit name is clearer for parallel agents.

## What is isolated

Each monavi task has:

- its own task ID and tab group;
- its own controlled tabs and active tab;
- its own refs and page state;
- independent approval and takeover state.

The user's browser shares cookies, localStorage, IndexedDB, cache, and login state across tabs. Monavi does not create a separate browser profile; that is intentional, because the task runs in the user's real browser.

## Parallel tasks

A device can run several tasks at once, each with its own tab group. Give each agent a distinct `--session` and do not operate on another task's tab. Close your own session when done.

```bash
monavi browser --session research open https://docs.example.com
monavi browser --session checkout open https://shop.example.com
```

## Inspect sessions

```bash
monavi browser session
monavi browser session list
monavi browser session info --json
```

`session` shows the current runtime session; `session list` lists runtime sessions. The monavi task is still identified by its task ID and is ended with `close`.

## Close the task

```bash
monavi browser close
```

Use the same `--session` name if you used one. Closing ends the task and lets the extension destroy the tab group immediately. The extension's **Keep tabs after a session ends** setting decides whether the tabs stay open; still close the session.

Do not leave a task open after answering the user. TTL is only a fallback, and an open task keeps the tab group alive until it expires.

## What not to use

Do not import cookies or browser state from files, and do not launch a separate browser profile. The user's existing browser session is the source of truth. If a page needs a login, ask the user to take over instead.
