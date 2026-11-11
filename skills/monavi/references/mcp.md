# monavi MCP

Use MCP when the host application already exposes MCP tools and you do not have a shell.

## Local stdio MCP

Start it with:

```bash
monavi mcp
# or pin the task / relay:
monavi mcp --task-id task-abc123 --relay https://relay.monavi.app
```

It exposes three tools:

- `monavi_repl` — a persistent JavaScript REPL with a typed `browser` API plus `ab()` as an escape hatch.
- `monavi_search_history` — opt-in browser history search.
- `monavi_search_bookmarks` — opt-in bookmark search. Try bookmarks before history.

`monavi_repl` also accepts `timeoutMs` (default 60s) and `maxOutputChars` (default 32 KiB, max 256 KiB) to bound text and logs. `structuredContent.value` is not truncated by `maxOutputChars`, so large objects remain available to machine-readable clients.

Use `browser` for structured results. It calls the bundled runtime with `--json` and returns the command's `data`, so a script can compose actions without parsing text:

```js
const page = await browser.open("https://example.com");
const snap = await browser.snapshot({ interactive: true });
print(snap.snapshot);

await browser.click("@e3");
await browser.wait.text("Saved");

const email = await browser.get.value("#email");
return { url: page.url, email };
```

Core methods cover `open`, `snapshot`, `click`, `fill`, `type`, `press`, `upload`, `download`, `find`, `wait.*`, `get.*`, `is.*`, `tab.*`, `frame`, `dialog.*`, `eval`, `read`, `screenshot`, `pdf`, `network.*`, `console`, `errors`, `batch`, `raw`, and `run`. Call `await browser.help()` for the full signatures and result examples. For any command not covered, use `browser.raw("<command>", ...args)`; `ab()` still returns raw stdout when you need it.

For a long script, check it without executing:

```js
const report = await browser.typecheck(`
  const s = await browser.snapshot();
  return s.snapshot;
`);
print(report);
```

Or set `typecheck: true` on `monavi_repl`; the tool checks the `code` argument before execution. Diagnostics use TypeScript codes and line/column positions. The checker is off by default, does not create a task, and treats persistent bare variables as `any`.

`browser.batch` runs several commands in one runtime process:

```js
const results = await browser.batch([
  ["open", "https://example.com"],
  ["snapshot", "-i"],
  ["click", "@e3"],
]);
```

`browser.session(name)` creates a named session with its own task, controlled tabs (or tab group), and approval. The default `browser` is the `default` session:

```js
const research = browser.session("research");
await research.open("https://example.com");
print(await research.task.status());
await research.task.close();
```

`browser.sessions()` lists sessions; `browser.closeAll()` closes all of them. Each named session is a separate task, so the user may need to approve it separately. Close every session when the work is done.

Keep one session/task open for the whole piece of work. Reuse the same `browser.session(name)` for every step, and call `browser.task.close()` once at the end. Closing between steps ends the task and the next call creates a new one. Prefer `browser.open` for sequential navigation; use `browser.tab.new` / `browser.tab.*` when pages must stay open in parallel.

The process can own multiple tasks through sessions. Set `MONAVI_TASK_ID` (or `--task-id`) for a stable base task ID; a new agent session should use a new ID. The process closes its used sessions when it exits. Raw `close` is blocked inside the REPL; use `browser.task.close()`.

`browser.*` throws `BrowserCommandError` with `code`, `message`, `command`, `exitCode`, and `data`. Common codes include `TASK_PAUSED`, `APPROVAL_DENIED`, `TASK_ENDED`, `TASK_CLOSED`, and `BROWSER_COMMAND_FAILED`.

## Remote MCP

Connect to `https://mcp.monavi.app/mcp` with a bearer token and an `X-Monavi-Task-Id` header. The remote server exposes `monavi_cdp` and task management. Use raw CDP for page actions; the server does not provide high-level page actions. Keep the same `X-Monavi-Task-Id` for every request in one agent session; a new ID creates a new task.

## Tool selection

- Need a site name resolved to a URL: bookmarks first, then history.
- Need page actions inside an MCP client: call `browser.*` inside `monavi_repl`; use `browser.help()` for the API and `browser.raw()` for commands outside the typed core.
- Need several independent tasks in one MCP process: `browser.session(name)`.
- Need raw CDP from a custom agent: `monavi_cdp`.
- Need to pause for the user: `monavi_takeover`; only the user can resume.
