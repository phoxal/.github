# CLAUDE.md — Claude Code / Claude API entry point

You are Claude, working in a Phoxal repository.

**Read these in order:**

1. [`docs/AI_ASSISTANT_GUIDE.md`](../docs/AI_ASSISTANT_GUIDE.md) — the
   canonical, cross-repo rules for any AI agent in any Phoxal repo.
2. The local `AGENTS.md` at the root of this repository — that repo's
   specific rules.
3. The local `README.md` for the crate / sub-tree you're editing.

The org-level guide is the rule unless the repo-local `AGENTS.md` overrides
it for that repo's specifics.

## Claude-specific notes

- **Use the right tool.** Prefer `Read`, `Edit`, `Write` over `cat`, `sed`,
  `echo` via `Bash`. Use `Bash` only for shell-only operations (git, builds,
  scripts).
- **Plan with `TaskCreate`/`TaskUpdate`** for non-trivial multi-step work,
  not in chat narration. Mark each task completed as soon as it's done.
- **Subagents.** Use `Explore` for broad codebase research; spawn focused
  agents in parallel when work is genuinely independent. Don't duplicate
  what a subagent is doing.
- **codex exec dispatch.** In `phoxal/robot-framework` and similar
  control-plane repos, the convention is reviewer-agent + `codex exec`
  writer-agent: you write the prompt, codex writes the code, you review and
  verify. See the control repo's `CLAUDE.md` for the canonical invocation.
- **Verification.** UI/Rerun/Webots changes require a visual check per the
  owning repo's `MACOS_VISUAL_VERIFICATION.md` or
  `RERUN_WEBOTS_VISUAL_VERIFICATION.md` runbook. If you cannot verify
  visually, say so explicitly — do not claim success.

## What not to do

- Don't introduce backwards-compat shims for in-flight v0.x contracts.
- Don't synthesize per-runtime config files.
- Don't add runtime introspection protocols (`--phoxal-describe`,
  `RuntimeDescriptor`).
- Don't push to default branches without explicit user direction.

See the canonical guide for the full list.
