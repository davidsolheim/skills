---
name: agent-onboard-prompt
description: >
  Write a paste-ready onboarding or goal prompt for another computer or
  agent. Use when the user runs /agent-onboard-prompt, says "write a goal
  prompt for me to use on another agent", "write an onboarding prompt for
  the other computer", or "give me a prompt i can give to dts-1 to get
  Notion and .wcp/issues/ onboarded like you have here."
user-invocable: true
---

# /agent-onboard-prompt — Prompt for another machine

Write a single prompt the user can paste on another computer or agent.
Do the research here. The other agent should not need this session.

## Trigger phrases

`/agent-onboard-prompt`, `write a goal prompt for me to use on another agent`, `write an onboarding prompt for the other computer`, `give me a prompt i can give to dts-1 to get Notion and .wcp/issues/ onboarded like you have here.`

## Steps

1. Name the target machine or agent and the job (MCP, CLI, repo, Notion, `.wcp/issues/`, and so on).
2. Read how this machine already does that job: user skills, `config.toml` **names only**, docs under `~/.grok/docs`, repo `AGENTS.md`. Never paste tokens, headers, Doppler values, or auth files.
3. Write one prompt with: objective, exact commands, file paths, success checks, and what to ask the human if blocked.
4. The stock example onboards Notion and `.wcp/issues/`. If they name another tool, still write that prompt. Do not teach Linear onboarding as the default job.
5. Output only the paste-ready prompt plus a one-line how-to (which machine, which tool). Do not run the prompt here unless asked.

## Do not

- Do not file an issue instead of writing the prompt (`/issue` is a different job).
- Print secrets. Use env **names** and "read from Doppler / existing MCP config" only.
