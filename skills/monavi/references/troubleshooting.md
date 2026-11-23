# monavi troubleshooting

## Setup and runtime

| Symptom | Action |
|---|---|
| `AUTH_REQUIRED` | Run `monavi setup <pairing-code>`. Generate a fresh code in the monavi extension popup. |
| `BROWSER_RUNTIME_MISSING` | Reinstall the CLI: `npm i -g @zenlix/monavi`. |
| `monavi doctor` reports a missing or stale plugin | Run `monavi setup --reset`, then pair again with a fresh code. |
| `monavi` is not found | Install it globally: `npm i -g @zenlix/monavi`. If you use pnpm, pass the version explicitly; `minimumReleaseAge` may otherwise pick an older release. |
| `EPERM` writing under `~/.monavi` | monavi needs a writable `MONAVI_HOME` (default `~/.monavi`) for credentials and session state. Report the permission error and ask the user to fix it; this is an environment problem, not a reason to switch tools. |
| Relay unreachable | Check the device connection in the extension popup and the relay URL in `monavi doctor`. |
| Not sure whether this machine is paired | Run `monavi status`; add `--json` for scripts. |
| Cannot find a running session | Run `monavi sessions` to list tracked sessions and their task status. |

## Task lifecycle

| Symptom | Action |
|---|---|
| `APPROVAL_TIMEOUT` | The relay rejected the pending task token or the task expired. Make sure the relay is current, ask the user to approve, then retry with the pinned CLI. |
| `APPROVAL_DENIED` | The user declined. Stop; do not retry the same task. |
| A click opens a new browser window | Popup windows and new windows are outside the task. Ask the user to open the URL in the current controlled tab, or run `monavi browser tab new <url>`. |
| A click appears to do nothing | The link may have opened a new tab. Run `monavi browser tab`, switch with `tab tN`, then snapshot the new page. |
| `TASK_PAUSED` | The task is paused. Keep the task, ask the user to resume in the page banner or the monavi extension popup, then snapshot again before continuing. |
| `TASK_ENDED` | The task is over. Start a new session with a new task ID. |
| `TASK_NOT_FOUND` | The task expired or was stopped. Start a new session. |
| `SESSION_EXPIRED` / `SESSION_NOT_RUNNING` | The saved session's task is no longer usable. The CLI already stopped the stale browser session; run `monavi browser open <url>` to start a new one. |
| `Plugin 'monavi' returned success=false` | The browser runtime hid the real error. The CLI follows up with a `TASK_EXPIRED` line when the task is over; run `monavi browser open <url>`. If no hint appears, run `monavi report` and share the output with the user. |
| A new task starts on every step | The previous task was closed or the session name changed. Reuse the same `--session` / `MONAVI_TASK_ID` and call `close` once at the end. |
| `Failed to read: Resource temporarily unavailable` | The session was created by an older CLI whose approval wait exceeded the browser runtime's 30s limit. Upgrade to `@zenlix/monavi@latest`, run `monavi browser close`, then retry the same command. |
| Tab group stays after the task | Run `monavi browser close` with the same `--session`. If it still stays, report it with the task ID. |

## Browser mechanics

| Symptom | Action |
|---|---|
| `Element not found: @eN` or stale ref | The page changed. Run `snapshot -i` again and use the new refs. |
| Click reports the element is covered | Dismiss the covering element, snapshot again, then retry. |
| Fill or type has no effect | Focus the element, then use `keyboard inserttext "text"` or `keyboard type "text"`. |
| Element is in the DOM but not in the snapshot | It is off-screen or not rendered yet. Scroll or `wait --text`, then snapshot. |
| `page_wedged` | The page stopped responding. Navigate again or close the task if it does not recover. |
| Cross-origin iframe missing from snapshot | Switch with `frame` if possible; otherwise ask the user to take over. |
| Dialog blocks the page | Use `dialog status`, then `dialog accept` or `dialog dismiss`. |
| Tab switched but refs fail | Refs belong to the active tab. Re-snapshot after switching. |
| Navigation is blocked | The page tried to leave the approved http(s) scope. Ask the user whether to continue in a new task. |
| Overlay is unavailable | Some browser-internal pages cannot be controlled. Ask the user to take over or use another URL. |

## Browser context and diagnostics

| Symptom | Action |
|---|---|
| `BOOKMARKS_DISABLED` | Ask the user to enable Bookmarks in the monavi extension popup. |
| `HISTORY_DISABLED` | Ask the user to enable History in the monavi extension popup. |
| `RATE_LIMITED` | Bookmarks and history share a rate limit. Wait before retrying. |
| `DEVICE_OFFLINE` | Ask the user to check that the browser and the extension are connected. |
| `CDP_METHOD_BLOCKED` | The extension blocked a sensitive method. Treat it as a policy decision; do not retry the same call. |
| Console or network list is empty | Events are captured only after the session enabled them. Navigate or interact, then run the command again; use `--clear` for a clean baseline. |
| `a11y` or `vitals` fails | The page may block injected scripts or lack the required APIs. Use `snapshot`, `eval`, or ask the user to take over. |
| Screenshot is missing | Check the path and permissions; use `monavi browser screenshot <path>` explicitly. |

## Reporting a problem

Include the monavi command, task ID (if known), error code, and what the user expected. Do not paste page content that may contain secrets.
