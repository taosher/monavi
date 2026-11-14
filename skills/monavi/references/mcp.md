# monavi MCP

Use MCP when the host application exposes MCP tools and you do not have a shell.

## Local stdio MCP

Start it with:

```bash
monavi mcp
# or pin the task / relay:
monavi mcp --task-id task-abc123 --relay https://relay.monavi.app
```

It exposes three tools:

- `monavi_repl` — a persistent Playwright REPL for the user's Chrome.
- `monavi_search_history` — opt-in browser history search.
- `monavi_search_bookmarks` — opt-in bookmark search. Try bookmarks before history.

`monavi_repl` also accepts `timeoutMs` (default 60s) and `maxOutputChars` (default 32 KiB, max 256 KiB) to bound text and logs. `structuredContent.value` is not truncated by `maxOutputChars`, so large objects remain available to machine-readable clients.

### Start a task

The first `open()` creates a task and asks the user to approve it in the monavi extension. Keep the same task for the whole piece of work:

```js
page = await open("https://example.com");
return await page.title();
```

`open(url?)` returns a Playwright `Page`. Assign it to `page` (a bare assignment persists across calls); `page`, `context`, and `browser` are available in later calls. A `const`/`let` only lives in one call.

### Read the page

Use Playwright locators and the accessibility snapshot. The snapshot is compact and is usually the best way to decide the next action:

```js
const snapshot = await page.locator("body").ariaSnapshot();
print(snapshot);

const heading = await page.getByRole("heading").first().textContent();
const links = await page.getByRole("link").allTextContents();
return { heading, links };
```

### Interact

Use standard Playwright calls. Auto-waiting handles most timing:

```js
await page.getByRole("button", { name: "Sign in" }).click();
await page.getByLabel("Email").fill("user@example.com");
await page.getByLabel("Password").fill("secret");
await page.getByRole("button", { name: "Continue" }).click();
await page.waitForURL("**/dashboard");
```

More examples:

```js
await page.getByText("Save").click();
await page.locator("#invoice-table tr").nth(2).click();
await page.keyboard.press("Enter");
await page.getByLabel("Country").selectOption("CN");
await page.getByRole("checkbox", { name: "Remember me" }).check();
await page.getByRole("button", { name: "Upload" }).setInputFiles("/tmp/report.pdf");
```

### Wait and inspect network

```js
const response = page.waitForResponse((res) => res.url().includes("/api/items") && res.ok());
await page.getByRole("button", { name: "Refresh" }).click();
await response;

const requests = [];
page.on("request", (request) => requests.push(request.url()));
await page.getByRole("button", { name: "Load more" }).click();
await page.waitForLoadState("networkidle");
```

### Evaluate in the page

```js
const total = await page.evaluate(() => document.querySelectorAll("tr").length);
const text = await page.evaluate((selector) => document.querySelector(selector)?.textContent, "#total");
return { total, text };
```

### Multiple tabs

`context.newPage()` creates another controlled tab in the same task. `page.close()` closes only that tab.

```js
const second = await context.newPage();
await second.goto("https://example.com/docs");
const title = await second.title();
await second.close();
return title;
```

### Multiple tasks

`monavi.session(name)` creates or reuses a named session with its own task, tab group, approval, and Playwright objects. Use it when two pieces of work must stay independent:

```js
const research = monavi.session("research");
const researchPage = await research.open("https://example.com");
await researchPage.getByRole("link", { name: "Docs" }).click();

const invoice = monavi.session("invoice");
const invoicePage = await invoice.open("https://example.com/invoices");
return {
  research: await researchPage.title(),
  invoice: await invoicePage.title(),
};
```

Close named sessions when they are done:

```js
await monavi.session("research").close();
```

### Screenshot

```js
return await screenshot({ fullPage: true });
```

`screenshot()` writes a temporary file and also returns the image to the MCP client.

### Finish the task

Always close the task when the work is done:

```js
await monavi.end();
```

`monavi.end()` closes the Playwright connection and ends the default task. `monavi.task.status()` and `monavi.task.close()` are the same for the default session. `monavi.closeAll()` ends every session in the process.

### Type checking

Check a long script before executing it:

```js
const report = await monavi.typecheck(`
  page = await open("https://example.com");
  await page.getByRole("button", { name: "Sign in" }).click();
`);
print(report);
```

Or set `typecheck: true` on `monavi_repl`. Diagnostics use TypeScript codes and line/column positions. The checker does not execute the code and does not create a task.

### Errors and takeover

`monavi_repl` returns `{code, message}` errors. Common codes:

- `TASK_PAUSED` — the user is controlling the tab. Keep the task, ask the user to click **Resume**, then retry the same action.
- `APPROVAL_DENIED` — the user denied the task. Do not retry the same task.
- `APPROVAL_TIMEOUT` — the user did not approve in time. Ask them to approve, then retry.
- `TASK_ENDED` — the task is over (stopped, expired, or closed). Create a new task with a new session name or restart `monavi mcp`.
- `TASK_CLOSED` — this session was closed. Use a new session name.
- `AUTH_REQUIRED` — run `monavi setup <pairing-code>` first.
- `RELAY_UNREACHABLE` — the relay is not reachable from this machine.

### Playwright support

The REPL runs Playwright against the user's current browser context. It supports navigation, locators, auto-wait, evaluation, dialogs, uploads, keyboard/mouse, network events, screenshots, PDFs, multiple tabs, and same-origin iframes.

It does not create new browser contexts or export cookies. Browser-level storage, permission, download, popup, service-worker, and cross-origin iframe APIs are outside monavi's task model. `browser.close()` only closes the Playwright client; use `monavi.end()` to end the task.

## Remote MCP

Connect to `https://mcp.monavi.app/mcp` with a bearer token and an `X-Monavi-Task-Id` header. The remote server exposes `monavi_cdp` and task management. Use raw CDP for page actions; the server does not provide a Playwright REPL. Keep the same `X-Monavi-Task-Id` for every request in one agent session; a new ID creates a new task.

## Tool selection

- Need a site name resolved to a URL: bookmarks first, then history.
- Need page actions inside a local MCP client: use `monavi_repl` with Playwright.
- Need several independent tasks in one MCP process: `monavi.session(name)`.
- Need raw CDP from a custom agent: `monavi_cdp`.
- Need to pause for the user: `monavi_takeover`; only the user can resume.
