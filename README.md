# monavi skill

Public distribution mirror of the `monavi` Agent skill. It teaches coding agents
how to use [monavi](https://monavi.app/) to drive the user's real browser through
`monavi browser`, with explicit user approval and takeover.

## Install

```bash
npx skills add taosher/monavi --skill monavi -g
```

## Requirements

Install the monavi CLI before the agent uses the skill:

```bash
npm i -g @zenlix/monavi
monavi setup <pairing-code>
```

Generate the one-time pairing code from the monavi extension popup. After pairing,
the agent can create browser tasks; each new task still requires approval in Chrome.

## Source of truth

The skill ships with `@zenlix/monavi` and is served by `monavi skills get monavi`,
always matching the installed CLI version. This repository mirrors
`apps/cli/skill-data/monavi` from the monavi project for `skills.sh` discovery.
If the two disagree, the CLI version wins.
