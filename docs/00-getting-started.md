# Getting Started with Spur

Welcome to Spur! This guide will walk you through installing the Spur TUI, initializing your first workspace, and orchestrating your first agent-driven task.

## 1. Prerequisites & Installation

Before using Spur, you need to ensure you have the underlying backend AI agents installed, as Spur acts as the orchestrator (the "Brain") that delegates tasks to these tools.

### Prerequisites

- **Git Repository:** Spur is designed to run inside a git repository.
- **Backend Agents:** You must have at least one supported AI CLI agent installed and authenticated on your system (e.g., [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) or Codex). Ensure the agent's executable (like `claude`) is available in your system's `$PATH`.

### Installation

Spur is a Rust-native application delivered conveniently via an npm wrapper package. You can install it globally or run it directly using `npx`.

**Install globally via npm:**

```bash
npm install -g @getspur/spur-cli
```

**Or run directly via npx:**

```bash
npx @getspur/spur-cli tui
```

<iframe title="vimeo-player" src="https://player.vimeo.com/video/1189052932?h=a02f66f3d8" width="640" height="360" frameborder="0" referrerpolicy="strict-origin-when-cross-origin" allow="autoplay; fullscreen; picture-in-picture; clipboard-write; encrypted-media; web-share"   allowfullscreen></iframe>

## 2. Initialization

Spur keeps its configuration and session data isolated per repository in a `.spur` directory.

To get started, navigate to the root of your project repository in your terminal and run:

```bash
spur init
```

When you run `spur init`, the CLI performs several intelligent setup steps:

- **Agent Discovery:** Scans your system's `$PATH` for supported agents (like `claude`, `codex`, `gemini`, or `kiro`) and registers them in your config. If an agent is missing, it provides a handy installation hint (e.g., `npm install -g @anthropic-ai/claude-code`).
- **Project Management Tools:** Checks for optional local PM tools like `br` (beads) or `bv` for tracking issues.
- **Interactive Setup:** Prompts you for optional setups, such as configuring a Telegram bot for remote interaction, and explicitly asks you to review permission bypasses (auto-approvals) for safety.
- **Configuration Generation:** Creates or merges `.spur/config.toml` with your discovered agents, setting one as the "Brain" orchestrator. _(Note: `spur init` is safe to run multiple times; it will merge new agents without overwriting your manual customizations.)_

_(Tip: Run `spur init --skills` to additionally install the bundled SpurPower agent skills to your repository.)_

## 3. First Launch

Once initialized, start the Spur TUI by running:

```bash
spur
```

_(Or use `npx @getspur/spur-cli tui` if you didn't install it globally)._

You will be greeted by the Spur terminal user interface, which displays your active sessions, agent status, and conversation history.

<iframe title="vimeo-player" src="https://player.vimeo.com/video/1189052932?h=a02f66f3d8" width="640" height="360" frameborder="0" referrerpolicy="strict-origin-when-cross-origin" allow="autoplay; fullscreen; picture-in-picture; clipboard-write; encrypted-media; web-share"   allowfullscreen></iframe>

## 4. First Steps

Now that Spur is running, it's time to orchestrate your first task!

1. **Create a Session:** If not already in an active session, follow the on-screen keybinds to start a new one.
2. **Enter a Prompt:** Use the input box at the bottom to type your request. Be as specific as you like. For example:
   - _"Add a dark mode toggle to the navigation bar."_
   - _"Refactor the authentication module to use the new database schema."_
   - _"Fix the memory leak in the worker pool."_
3. **Watch the Orchestration:** Once you submit the prompt, the **Brain** agent takes over. It will analyze your request, formulate an implementation plan, and delegate the execution to a **Worker** agent (like Claude Code) with your prompt explicit. You can sit back and monitor the real-time execution, logs, and tool calls directly from the TUI.

> 🎥 **Video Placeholder:** Executing your first task and watching delegation.

Congratulations! You are now using Spur to manage and orchestrate AI development tasks in your workspace.
