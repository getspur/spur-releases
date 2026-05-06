# Permissions & Bypass

Spur sits between you and your agents. Every tool an agent wants to run — `bash`, `write_file`, `web_fetch` — flows through Spur's permission gate. By default that gate prompts you ("Agent X wants to run command Y — allow?"). For long-running autonomous workflows, you can configure per-agent bypass so trusted agents act without per-call confirmation.

This page explains how the permission system works, the three levers you can turn, and the safety guardrails Spur builds in.

> **Security notice.** Bypass means the agent runs tools without confirming with you first. Set it only on agents you trust, in repositories where you're prepared for autonomous changes, and only after reading [§ Security implications](#security-implications) below.

---

## How permissions flow

```
┌─────────────┐   tool call    ┌─────────────┐   ask user?   ┌─────┐
│ Agent (ACP) │ ─────────────► │  Spur gate  │ ────────────► │ You │
└─────────────┘                └─────────────┘   y / n        └─────┘
                                      │
                                      └── if bypass on for this agent,
                                          gate auto-approves and the
                                          tool runs without prompting
```

Two layers cooperate:
- **L1 — Agent-side bypass.** The agent never asks Spur in the first place. Spawn-time CLI flags (`--dangerously-skip-permissions`) or ACP `new_session` modes (`session_mode = "bypassPermissions"`) tell the agent itself to skip its own confirmation UI.
- **L2 — Spur-side auto-approve.** When Spur receives a permission-prompt request from the agent (some agents always ask via ACP), Spur can auto-respond *yes* if the agent is configured for bypass.

Most agents use one or the other; a few use both. The three config levers below let you express either or both.

---

## The three levers

All three live under `[agents.entries.permissions]` in `.spur/config.toml`:

```toml
[[agents.entries]]
name = "my-agent"
# ...

[agents.entries.permissions]
skip         = true                         # master switch
args         = ["--dangerously-skip-permissions"]
session_mode = "bypassPermissions"
```

| Lever | Type | Purpose |
|---|---|---|
| `skip` | `bool` | Master switch. When `false` (default), Spur always prompts. When `true`, Spur enables L2 auto-approve and applies the other levers below. |
| `args` | `Vec<String>` | Spawn-time CLI args appended to the agent's `args` when `skip = true`. Use for CLI agents that bypass via flags (most non-ACP agents). |
| `session_mode` | `Option<String>` | A string passed to the agent in the ACP `new_session` request when `skip = true`. Use for ACP agents that bypass via session metadata (Claude Code, etc.). |

**At least one of `args` or `session_mode` should be set when `skip = true`.** Without an explicit mechanism, Spur falls back to L2 auto-approve only — the agent will keep popping its own permission prompt and Spur will silently dismiss it, which works but is fragile across agent versions.

---

## Per-agent recipes

### Claude Code ACP — session-mode bypass

Claude Code's ACP server reads a `permission_mode` field on `new_session`. Spur sends whatever you put in `session_mode`.

```toml
[[agents.entries]]
name = "claude-code"
command = "npx"
args = ["--yes", "@agentclientprotocol/claude-agent-acp@latest"]
transport = "acp"

[agents.entries.permissions]
skip = true
session_mode = "bypassPermissions"
# args = []  # not needed for Claude Code
```

Acceptable values for `session_mode` (per Claude Code's ACP server): `"default"`, `"acceptEdits"`, `"bypassPermissions"`, `"plan"`. `"bypassPermissions"` is the strongest.

### Codex / generic CLI — args bypass

Codex (and most pure-CLI agents) bypass via a flag at spawn time.

```toml
[[agents.entries]]
name = "codex"
command = "npx"
args = ["@zed-industries/codex-acp"]

[agents.entries.permissions]
skip = true
args = ["--dangerously-skip-permissions"]
```

Spur appends `args` to the agent's existing `args` only when `skip = true`. With `skip = false`, the flag is omitted and the agent runs in its normal interactive-prompt mode.

### Gemini CLI — also args-based

```toml
[[agents.entries]]
name = "gemini"
command = "gemini"

[agents.entries.permissions]
skip = true
args = ["--yolo"]   # Gemini's bypass flag
```

### Both layers (belt-and-suspenders)

For agents that support both CLI flags and ACP session modes, set both. Spur applies whichever the agent honors.

```toml
[agents.entries.permissions]
skip = true
args = ["--dangerously-skip-permissions"]
session_mode = "bypassPermissions"
```

### Mixed fleet — bypass some agents, prompt others

A common pattern: trust your worker agents to act, but keep human-in-the-loop on the brain.

```toml
[[agents.entries]]
name = "claude-code"   # brain
[agents.entries.permissions]
skip = false           # always prompt — you're the gate

[[agents.entries]]
name = "codex"         # worker
[agents.entries.permissions]
skip = true
args = ["--dangerously-skip-permissions"]
```

---

## The init-time safety prompt

When you run `spur init` and any agent has `skip = true`, Spur surfaces a TTY warning before persisting:

```
WARNING: the following agents have permission bypass enabled:
  - claude-code
  - codex

Permission bypass allows agents to execute tools without prompting.
Keep bypass enabled? [y/N]
>
```

Pressing anything other than `y` flips `skip = false` for **every** agent and rewrites `.spur/config.toml`. The default-no answer is deliberate — you should re-affirm bypass each time you re-run `init`, since `init` is idempotent and may surface this prompt after a config change you didn't expect.

This prompt is TTY-only; non-interactive runs (CI, scripts) skip it and persist as-configured.

---

## Validator warnings

Spur runs a pre-persist validator on every `spur init` and surfaces issues by severity:

| Rule | When | Severity |
|---|---|---|
| **R3** | `skip = true` with empty `args` AND `session_mode = None` | WARN — "relying on L2 auto-approve only; consider setting `permissions.args` or `permissions.session_mode`" |

The R3 warning is non-fatal — your config still persists — but it's surfaced because the L2-only path has produced subtle "agent keeps prompting and nothing happens" bugs across past agent versions. Set an explicit `args` or `session_mode` to take L1 control.

You'll see the warning rendered like this on `spur init`:

```
[spur] config validation warning agent=kimi warning=kimi: permissions.skip = true with no explicit mechanism — relying on L2 auto-approve only; consider setting permissions.args or permissions.session_mode
```

---

## Legacy flat fields (still supported)

Earlier versions of Spur used three flat fields directly on the agent entry. They still work — Spur's `effective_permissions()` merges nested-into-flat with nested winning when set:

```toml
# Legacy shape (still parsed)
[[agents.entries]]
name = "old-agent"
skip_permissions = true
skip_permissions_args = ["--dangerously-skip-permissions"]
skip_permissions_session_mode = "bypassPermissions"
```

| Legacy field | Modern equivalent |
|---|---|
| `skip_permissions` | `permissions.skip` |
| `skip_permissions_args` | `permissions.args` |
| `skip_permissions_session_mode` | `permissions.session_mode` |

New configs should use the nested form. Legacy fields will be supported indefinitely but won't gain new levers.

---

## Security implications

Bypass is convenient but expensive if abused. Concrete risks:

- **`bash`-class tool calls run unattended.** The agent can `rm`, `git push --force`, exfiltrate via `curl`, or run package-install commands you'd otherwise reject.
- **`write_file` lands silently.** If the agent decides your `.env`, `~/.ssh/config`, or `~/.gitconfig` needs editing, it edits.
- **Worktree isolation does NOT contain `bash`.** Spur isolates *file writes* to the worktree, but `bash` calls inherit your full process environment. An agent inside a worktree can still touch your home directory, write to system paths, or talk to remote servers.

Practical guardrails:

1. **Don't bypass the brain.** Keep your orchestrator agent on `skip = false` so plan-level decisions go through you.
2. **Bypass workers in trusted repositories.** Workers do narrow, scoped tasks; the brain has already approved each delegation. Worker bypass is the normal high-throughput path.
3. **Audit `.spur/config.toml` in code review.** Any PR that flips `skip = true` deserves a second pair of eyes.
4. **Run risky agents inside Spur's worktree isolation only.** Even with bypass, file writes go into the ephemeral worktree, not your branch. Approve before merge.
5. **Disable bypass for shared / CI repositories.** Multi-user repos shouldn't auto-approve anything by default; force interactive review.

---

## Troubleshooting

**"Agent keeps asking for permission even though `skip = true`."**
You probably don't have an explicit mechanism set. Check for the R3 warning on the last `spur init` output. Add either `permissions.args` (CLI flag) or `permissions.session_mode` (ACP) — see the per-agent recipes above.

**"Bypass worked yesterday and now it doesn't."**
Agent updates sometimes change their CLI flags or ACP fields. Check the agent's release notes and update `permissions.args` / `permissions.session_mode` to match. Alternatively, run `spur init` to re-discover the agent's current preferences.

**"My CI run hung at a permission prompt."**
CI environments are non-TTY, so `spur init`'s safety prompt was skipped — but the agent itself may still be prompting. Make sure `permissions.skip = true` *and* an explicit mechanism is set for any agent your CI spawns.

**"I want bypass for one task, prompt for another."**
Keep `skip = true` on the agent and use the `--no-permissions` (or equivalent) flag on the specific run if the agent supports it. Or run the cautious task with a separate agent entry that has `skip = false`.

---

## See also

- [Configuration](05-configuration.md) — full `.spur/config.toml` reference
- [Getting Started](00-getting-started.md) — first-run walkthrough including the bypass prompt
- [Known Issues](06-known-issues.md) — current rough edges
