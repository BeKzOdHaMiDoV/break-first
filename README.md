# Break First

A plugin for [Claude Code](https://code.claude.com) that attacks your own code before attackers do. The main Claude session acts as an orchestrator: it plans the task and hands the work to specialized subagents that build, review, test, simplify and then attack the result.

> **Status: early development.** Expect breaking changes.

> Unofficial community project. Not affiliated with or endorsed by Anthropic.

## What's included

- **`/break-first:orchestrate`** (skill): sizes the task, does trivial changes itself, splits larger work into small steps for the coder, then has the reviewer and tester check the result. Review issues and failing tests go back to the coder (only the essentials), for at most 2 fix rounds. Once tests are green and review is done, the refactor agent proposes simplifications that the coder applies without changing behavior (tests are re-run to prove it). Then the red-team attacks the code; proven holes go back to the coder, and the red-team re-checks, for at most 2 security rounds before it reports back to you.
- **`break-first:coder`** (subagent): writes code straight into files, verifies it, and returns a short report (files touched, how to run, what was checked) instead of pasting code back.
- **`break-first:reviewer`** (subagent, read-only): reviews new code and large edits for logic errors, duplication, needless complexity, dead code and bloat. Its main question: did the code get better, not just bigger? Reports problem → file:line → how to simplify, or a single `OK` line.
- **`break-first:tester`** (subagent): writes unit tests against the expected behavior, runs them, and reports passed/failed counts with file:line and cause for each failure. It only touches test files, never project code.

- **`break-first:refactor`** (subagent, read-only): after the code works, looks at new or heavily changed code for duplication, dead code, needless complexity and overlong functions. Its one question: how to make the code smaller and clearer with exactly the same behavior? Reports place → problem → proposal, or a single `NOTHING TO SIMPLIFY` line.
- **`break-first:red-team`** (subagent): attacks only the current project's code, locally. First detects the project's stack and runs the free scanners that fit it, if they are already installed (for example `bandit`/`pip-audit`, `npm audit`, `semgrep`, OWASP `dependency-check`). It never installs a scanner itself: if one is missing, it tells you how to install it and goes on by hand. Then it always looks for logic holes scanners miss: access-check bypass, unsafe input, data leaks, injections. Every hole must be proven by an attack test that fails while the hole exists; anything unproven is reported as a hypothesis. It never edits project code.

Shared rules: save tokens, make code better rather than bigger, and touch nothing outside the task.

Security limits: the red-team attacks only the project from the task, never runs real network attacks or touches other systems, and never deletes or corrupts data (it works on copies and temporary files inside the project). These limits are instructions to the model, not a sandbox; review what it runs, as with any agent.

## Languages

break-first is not tied to one language. It detects the project's stack and uses that stack's own tools:

- **Tests:** the test framework the project already uses, otherwise the language's standard one (for example pytest, Jest or Vitest, JUnit, go test).
- **Build and checks:** the project's own build, run and lint commands.
- **Security scanners:** free scanners that fit the stack, if installed (for example bandit/pip-audit, npm audit, semgrep, OWASP dependency-check). With no scanner, the red-team relies on manual review and attack tests, which it does on every stack anyway.

If a tool is missing, no agent installs it silently: you get the install command, and the work continues with what is available.

So far the plugin has been tested live on Python projects.

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

The orchestrator runs on your session's model. The coder, reviewer, tester and refactor agents always run on Sonnet, and the red-team on Opus, whatever your session model is.

Sonnet is cheaper per token, but a cheaper orchestrator doesn't always mean a cheaper run. On tasks with an ambiguous spec, an Opus orchestrator tends to write more precise briefs and saves fix rounds.

In our test (one run each, same task: a console TODO list with JSON storage):

| Session model | Orchestrator | Subagents | Total | Fix rounds |
|---|---|---|---|---|
| Opus | $0.43 | $0.14 | $0.57 | 1 |
| Sonnet | ≈ $0.49 | ≈ $0.18 | $0.67 | 1, plus tests rewritten after a contradictory brief |

The Sonnet split is estimated from token usage, since both parts ran on the same model. One run per model is not a benchmark; your results will vary.

## License

[MIT](LICENSE)
