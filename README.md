# Break First

A plugin for [Claude Code](https://code.claude.com). The main Claude session acts as an orchestrator: it plans the task and hands implementation steps to specialized subagents that do the work.

> **Status: early development.** Expect breaking changes.

> Unofficial community project. Not affiliated with or endorsed by Anthropic.

## What's included

- **`/break-first:orchestrate`** (skill): sizes the task, does trivial changes itself, and splits larger work into small steps delegated to the coder.
- **`break-first:coder`** (subagent): writes code straight into files, verifies it, and returns a short report (files touched, how to run, what was checked) instead of pasting code back.

Shared rules: save tokens, make code better rather than bigger, and touch nothing outside the task.

## Install

In Claude Code:

```
/plugin marketplace add BeKzOdHaMiDoV/break-first
/plugin install break-first@break-first
```

Then run:

```
/break-first:orchestrate <your task>
```

## License

[MIT](LICENSE)
