# Break First

A plugin for [Claude Code](https://code.claude.com). The main Claude session acts as an orchestrator: it plans the task and hands implementation steps to specialized subagents that do the work.

> **Status: early development.** Expect breaking changes.

> Unofficial community project. Not affiliated with or endorsed by Anthropic.

## What's included

- **`/break-first:orchestrate`** (skill): sizes the task, does trivial changes itself, splits larger work into small steps for the coder, then has the reviewer and tester check the result. Review issues and failing tests go back to the coder (only the essentials), for at most 2 fix rounds before it reports back to you.
- **`break-first:coder`** (subagent): writes code straight into files, verifies it, and returns a short report (files touched, how to run, what was checked) instead of pasting code back.
- **`break-first:reviewer`** (subagent, read-only): reviews new code and large edits for logic errors, duplication, needless complexity, dead code and bloat. Its main question: did the code get better, not just bigger? Reports problem → file:line → how to simplify, or a single `OK` line.
- **`break-first:tester`** (subagent): writes unit tests against the expected behavior, runs them, and reports passed/failed counts with file:line and cause for each failure. It only touches test files, never project code.

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

## Cost

The orchestrator runs on your session's model. The subagents (coder, reviewer, tester) always run on Sonnet, whatever your session model is.

Sonnet is cheaper per token, but a cheaper orchestrator doesn't always mean a cheaper run. On tasks with an ambiguous spec, an Opus orchestrator tends to write more precise briefs and saves fix rounds.

In our test (one run each, same task: a console TODO list with JSON storage):

| Session model | Orchestrator | Subagents | Total | Fix rounds |
|---|---|---|---|---|
| Opus | $0.43 | $0.14 | $0.57 | 1 |
| Sonnet | ≈ $0.49 | ≈ $0.18 | $0.67 | 1, plus tests rewritten after a contradictory brief |

The Sonnet split is estimated from token usage, since both parts ran on the same model. One run per model is not a benchmark; your results will vary.

## License

[MIT](LICENSE)
