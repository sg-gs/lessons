# Coding with AI agents

Practical guidelines for working with coding agents (Claude Code, Codex). Both vendors converge on the same ideas: the agent is only as good as the context it gets, the checks it can run and the limits you set around it.

## 1. Let it run, but fence the field

Approving every command is safe on paper, but after the tenth prompt you are no longer reviewing anything, just clicking "yes". It is better to give the agent autonomy and put hard limits around it than to babysit each step.

- **Auto mode** (Claude Code): a classifier approves routine actions and blocks the risky ones (scope escalation, unknown infrastructure, actions driven by content it read).
- **Sandbox**: OS-level isolation of filesystem and network. The agent can do anything *inside* the fence and nothing outside it.
- **Permissions**: allowlist trusted commands and deny the dangerous ones explicitly (e.g. pushing to `main`, touching production checkouts).
- **Hooks**: for rules that must hold *every* time, don't ask nicely in a prompt, enforce them (e.g. run the linter after each edit, block writes to migrations).
- Codex has the same split: *approval mode* (when it asks) vs *sandbox mode* (what it can touch). Start with the defaults and loosen only for trusted repos.

## 2. Write a short, curated CLAUDE.md / AGENTS.md

A file loaded into every session with what the agent can't guess on its own.

- **Include**: build/test/lint commands, non-default style rules, repo etiquette (branches, commits, PRs), environment quirks, gotchas.
- **Exclude**: anything derivable from the code, standard language conventions, API docs (link them), file-by-file descriptions, info that changes often.
- For each line ask: *would removing it cause mistakes?* If not, cut it.
- Layer it: personal/global defaults, repo-level shared rules, subdirectory-specific rules.
- Treat it like code: version it, review it, prune it. When the agent repeats the same mistake twice, add a rule (or ask the agent for a retrospective and let it propose the rule).
- If a rule keeps being ignored, it probably belongs in a hook, not in the file.

## 3. Move repeated workflows into skills

If you keep pasting the same prompt or correcting the same workflow, turn it into a skill (`SKILL.md`).

- One skill, one job, with clear inputs and outputs.
- The description is what makes it trigger: say *what* it does and *when* to use it.
- Get the task working by hand on a real case first, then turn it into a skill.
- Domain knowledge goes in skills (loaded on demand), not in CLAUDE.md (loaded always).
- Workflows with side effects (deploys, opening PRs) should only run when invoked explicitly.

## 4. Always give it a way to verify, and always review

The single most impactful practice: the agent needs a check it can run by itself (tests, build, linter, screenshot). Without it, you become the verification loop.

- State what "done" means in the prompt: which tests pass, which behavior changes, which bug stops reproducing.
- Ask for evidence (test output, command results), not claims of success.
- Ask for root-cause fixes, not suppressed errors.
- Review the diff yourself before anything others see. A second, fresh session acting as reviewer (*writer/reviewer* pattern) catches what the author session is blind to. Keep the review focused on correctness and requirements: reviewers over-report, and acting on every nitpick leads to over-engineering.

## 5. Explore, plan, then code

- For non-trivial tasks use plan mode: let it read the code and propose a plan naming files and flow, correct the plan, then implement.
- For big features, let the agent interview you, write a spec, then start a fresh session to implement it.
- Skip planning when the change fits in one sentence (typo, rename, log line).
- A good prompt has: **goal**, **context** (files, errors, an existing pattern to copy), **constraints**, and **done when**.

## 6. Manage context like a scarce resource

Quality degrades as the context window fills up.

- One session per task; clear the context between unrelated tasks.
- If you've corrected it twice and it's still wrong, start over with a better prompt instead of a third correction.
- Delegate wide exploration to subagents so file dumps don't pollute the main thread.
- Run parallel agents in separate git worktrees so they never edit the same files.

## Common mistakes

- Mixing unrelated tasks in one long session.
- Durable rules repeated in prompts instead of CLAUDE.md, a skill or a hook.
- A bloated CLAUDE.md that the agent ends up ignoring.
- Trusting output without a check it could run.
- Granting full permissions before understanding the workflow, or no autonomy at all and approving everything blindly.
- Watching the agent step by step instead of working on something else in parallel.

## Sources

- Anthropic: [Claude Code best practices](https://code.claude.com/docs/en/best-practices), [Engineering blog post](https://www.anthropic.com/engineering/claude-code-best-practices)
- OpenAI: [Codex best practices](https://learn.chatgpt.com/codex/learn/best-practices)
