# Show HN: Spur – A Rust TUI that orchestrates AI coding agents in parallel git worktrees

**Issue in, PR out — across every agent.**

Hey HN,

For the past while, we’ve been frustrated by the single-threaded nature of CLI AI coding agents. When you ask an agent to build a feature, it takes over your terminal, mutates your working directory, and blocks you from doing any parallel work. If it hallucinates, your `git diff` is a mess. If you want two agents to work on different files, you get merge conflicts.

We built **Spur** to solve this. Spur is a Rust-native Terminal User Interface (TUI) and orchestration engine (~110k lines of Rust) that acts as a hypervisor for AI agents using the Agent Client Protocol (ACP) and Model Context Protocol (MCP).

Instead of letting agents touch your main branch directly, Spur delegates tasks into **ephemeral, isolated git worktrees**, manages dependencies between tasks via a DAG, and merges them back deterministically.

---

### 📸 A Look at the TUI

<iframe title="vimeo-player" src="https://player.vimeo.com/video/1189025066?h=b23bba16b9" width="640" height="360" frameborder="0" referrerpolicy="strict-origin-when-cross-origin" allow="autoplay; fullscreen; picture-in-picture; clipboard-write; encrypted-media; web-share"   allowfullscreen></iframe>

<iframe title="vimeo-player" src="https://player.vimeo.com/video/1189025066?h=b23bba16b9" width="640" height="360" frameborder="0" referrerpolicy="strict-origin-when-cross-origin" allow="autoplay; fullscreen; picture-in-picture; clipboard-write; encrypted-media; web-share"   allowfullscreen></iframe>

---

## ⚡ The Killer Feature: Worktree Isolation & G-Strict Merging

The biggest challenge with autonomous agents is state management. Spur solves this with what we call **Worktree Isolation Discipline**.

When you ask Spur to execute an Epic, the "Brain" orchestrator (e.g., Claude Code, Codex) breaks the work down into a dependency graph (DAG) of tasks. Here is what Spur does under the hood:

1. **Isolates:** For each ready task, Spur runs `git worktree add` to create a hidden, temporary directory.
2. **Synchronizes:** It safely inherits untracked files (like `.env` or `node_modules`).
3. **Overlays:** If Task B depends on Task A, Spur cherry-picks Task A's commits into Task B's worktree _before_ dispatching.
4. **Chroots:** The worker agent runs entirely inside this worktree.
5. **G-Strict Merge:** Once the Brain (or you) approves the task, Spur extracts the diff, performs a deterministic topological merge to `Main`, and deletes the worktree.

> See [Issues & Planning](https://github.com/getspur/spur-releases/blob/main/docs/04-issues-and-planning.md) for the Issue Browser, Plan Inspector, and DAG-driven execution UI in action.

---

## 🧠 Under the Hood (Architecture)

Spur isn't just a pretty UI wrapper; it's a heavy-duty orchestration engine built across 13 strict Rust crates (zero dependency cycles). We had to solve complex concurrency and state problems to make multi-agent orchestration safe.

### Simplified System Flow

Here is the beautifully simple workflow Spur enables—abstracting away the complex DAG reconcilers and event buses so you can focus on building:

```mermaid
graph TD
    classDef tui fill:#1a1a2e,stroke:#e94560,stroke-width:2px,color:#fff;
    classDef brain fill:#e94560,stroke:#fff,stroke-width:2px,color:#fff;
    classDef worker fill:#533483,stroke:#fff,stroke-width:2px,color:#fff;
    classDef worktree fill:#238636,stroke:#2ea043,stroke-width:2px,color:#fff;
    classDef repo fill:#0f3460,stroke:#113d8f,stroke-width:2px,color:#fff;

    User((User)) -->|"1. Build this feature"| Spur[Spur TUI]:::tui
    Spur -->|"2. Plans tasks"| Brain{Brain Agent}:::brain

    subgraph Invisible to Main Workspace
        direction TB
        WT1[Isolated Worktree 1<br/>Task A]:::worktree -.->|Writes Code| Worker1(Worker Agent):::worker
        WT2[Isolated Worktree 2<br/>Task B]:::worktree -.->|Writes Code| Worker2(Worker Agent):::worker
    end

    Brain -->|"3. Dispatches"| WT1
    Brain -->|"3. Dispatches"| WT2

    Worker1 -->|"4. Code Approved"| Merge{Deterministic Merge}:::tui
    Worker2 -->|"4. Code Approved"| Merge

    Merge -->|"5. Clean Commit"| Repo[(Your Main Branch)]:::repo
```

### Architectural Highlights

- **Pure Event Sourcing:** The TUI state (Lineage, Plan Inspector) is a pure projection of an NDJSON event stream (`EventFunnel`). Resuming a session is just a fast replay.
- **Worktree Authority (Lease-Aware GC):** To prevent orphaned worktrees if the app crashes, we implemented a `WorktreeAuthority` that uses `fs4` advisory locks as cross-process liveness probes. It runs periodic background sweeps (15 min + jitter) to reap dead worktrees safely.
- **Single-Attach Invariants:** We use kernel-level lockfiles (`.spur/sessions/<id>.lock`) to ensure you can't have two TUI windows sending prompts to the same brain session (split-brain).
- **Embedded Issue Store:** Issue tracking links the BEADS SQLite engine directly Per-call latency dropped from ~50 ms (spawn + JSON roundtrip) to <1 ms. Cross-process correctness is a documented contract: N concurrent readers across any spur instances + one writer machine-wide, via OS flock on `.beads/.write.lock` plus SQLite WAL snapshot isolation, with jittered exponential backoff under contention. Multi-process integration tests pin the contract

## 🚀 Getting Started

Spur operates locally in your repository, with issue tracking handled either by an embedded SQLite store

No signup required, run it directly in your repo via `npx`:

```bash
# In any git repository
npx @getspur/spur-cli tui

# Or install globally
npm install -g @getspur/spur-cli
spur init
spur tui
```

<iframe title="vimeo-player" src="https://player.vimeo.com/video/1189052932?h=a02f66f3d8" width="640" height="360" frameborder="0" referrerpolicy="strict-origin-when-cross-origin" allow="autoplay; fullscreen; picture-in-picture; clipboard-write; encrypted-media; web-share"   allowfullscreen></iframe>

When you run `spur init`, it automatically discovers agents installed on your `$PATH` (Claude Code, Aider, Codex, Gemini CLI) and sets them up in `.spur/config.toml`.

Type an issue into the input bar, hit Enter, and watch the Brain draw up a plan and spin up worktrees.

**Documentation** ([github.com/getspur/spur-releases](https://github.com/getspur/spur-releases)):

- [Getting Started](https://github.com/getspur/spur-releases/blob/main/docs/00-getting-started.md) — install, first run, the 60-second tour
- [Core Navigation](https://github.com/getspur/spur-releases/blob/main/docs/01-core-navigation.md) — panes, focus, keybindings
- [Session Management](https://github.com/getspur/spur-releases/blob/main/docs/02-session-management.md) — resuming, lineage, the event log
- [Commands & Input](https://github.com/getspur/spur-releases/blob/main/docs/03-commands-and-input.md) — slash commands, prompt routing
- [Issues & Planning](https://github.com/getspur/spur-releases/blob/main/docs/04-issues-and-planning.md) — Issue Browser, Plan Inspector, DAG execution
- [Configuration](https://github.com/getspur/spur-releases/blob/main/docs/05-configuration.md) — `.spur/config.toml`, brains, fallbacks, permissions

We'd love for the HN community to tear this apart, try out the worktree isolation, and tell us where the UX or the Rust architecture can be improved. I'll be hanging out in the comments all day to answer deep architectural or product questions!
