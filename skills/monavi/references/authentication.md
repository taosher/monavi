# Authentication and sensitive flows

monavi runs in the user's real browser, so the user's existing login state is already available. Do not import cookies, launch a separate profile, or ask the user to paste credentials.

## Normal login flow

When a page asks for a login, 2FA, captcha, or a judgment call:

1. Tell the user what you are trying to do.
2. Ask them to click **Take over** in the page banner.
3. Wait while they complete the login.
4. Ask them to click **Resume**.
5. Run `monavi browser snapshot -i` again before continuing.

While the task is paused, do not issue new commands. If a command was already in flight, it may return `TASK_PAUSED`; wait for the user to resume, then retry or snapshot again.

Do not try to guess credentials or bypass the login.

## Reusing a login

The task uses the user's existing browser session. After the user logs in once, later tasks in the same browser can reuse that session. Do not use `monavi browser state save` or a local auth-state file; monavi does not expose cookie/state import, and the user's browser is the source of truth.

## Auth vault

`monavi browser auth` is available for disposable development accounts, but prefer user takeover for real accounts. If you use it:

- pass the password with `--password-stdin`, never as a command argument;
- keep the profile name private;
- do not print, screenshot, or share the stored secret;
- delete the profile when it is no longer needed.

## Sensitive data

Passwords, one-time codes, session cookies, and tokens are not task output. Do not put them in messages, logs, screenshots, HAR files, or generated documents. If a page displays a secret, do not quote it back to the user.
