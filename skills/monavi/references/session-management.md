# Session and task management

A monavi task is one agent session with its own controlled tab (or tab group where the browser supports it). Use a stable session name and reuse it for every command in that task. Keep one task open for the whole piece of work; a task is the unit of work, not a single command.

## Name the session

```bash
monavi browser --session invoice-check open https://example.com
monavi browser --session invoice-check snapshot -i
monavi browser --session invoice-check close
```

If `MONAVI_TASK_ID` is set, use it as the task identity. Keep it stable for the whole session and start a new ID for a new session.

`close` is the final action of the task. Keep the task open across `open`, `snapshot`, `click`, and `wait`; closing between steps ends the task, and the next command creates a new task with a new controlled tab.

Monavi also derives a session from the working directory when you do not pass `--session`, but an explicit name is clearer for parallel agents.

## What is isolated

Each monavi task has:

- its own task ID and controlled tabs;
- its own controlled tabs and active tab;
- its own refs and page state;
- independent approval and takeover state.

The user's browser shares cookies, localStorage, IndexedDB, cache, and login state across tabs. Monavi does not create a separate browser profile; that is intentional, because the task runs in the user's real browser.

## Parallel tasks

A device can run several tasks at once, each with its own controlled tab or tab group. Give each agent a distinct `--session` and do not operate on another task's tab. Close your own session once when the whole task is done.

```bash
monavi browser --session research open https://docs.example.com
monavi browser --session checkout open https://shop.example.com
```

## Inspect sessions

```bash
monavi sessions
monavi browser session list
```

`monavi sessions` lists the sessions monavi has tracked for this machine, with their task status. `monavi browser session list` lists runtime sessions. The monavi task is still identified by its task ID and is ended with `close`.

## Close the task

```bash
monavi browser close
```

Use the same `--session` name if you used one. Closing ends the whole task and lets the extension clean up the task's tabs immediately. Keep the task open between steps; each new task creates a new controlled tab and may need a new approval. The extension's **Keep tabs after a session ends** setting decides whether the tabs stay open; still close the session once at the end.

Do not leave a task open after answering the user. TTL is only a fallback, and an open task keeps its tabs alive until it expires.

## What not to use

Do not import cookies or browser state from files, and do not launch a separate browser profile. The user's existing browser session is the source of truth. If a page needs a login, ask the user to take over instead.
