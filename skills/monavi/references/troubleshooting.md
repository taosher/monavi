# monavi troubleshooting

## Setup and runtime

| Symptom | Action |
|---|---|
| `AUTH_REQUIRED` | Run `monavi setup <pairing-code>`. Generate a fresh code in the monavi extension popup. |
| `BROWSER_RUNTIME_MISSING` | Reinstall the CLI: `npm i -g @zenlix/monavi@latest`. |
| `monavi doctor` reports a missing or stale plugin | Run `monavi setup --reset`, then pair again with a fresh code. |
| `monavi` is not found | Install it globally: `npm i -g @zenlix/monavi`. |
| Relay unreachable | Check the device connection in the extension popup and the relay URL in `monavi doctor`. |

## Task lifecycle

| Symptom | Action |
|---|---|
| `APPROVAL_TIMEOUT` | The user did not approve in time. Ask them to approve, then retry. |
| `APPROVAL_DENIED` | The user declined. Stop; do not retry the same task. |
| `TASK_PAUSED` | The user took over. Keep the task, wait for Resume, then snapshot again before continuing. |
| `TASK_ENDED` | The task is over. Start a new session with a new task ID. |
| `TASK_NOT_FOUND` | The task expired or was stopped. Start a new session. |
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
