# Spur

**Issue in, PR out — across every agent.**

Spur is a Rust-native Terminal User Interface (TUI) that orchestrates multiple AI coding agents through the Agent Client Protocol (ACP). A "brain" agent reasons about your task and delegates work to the best-fit worker agent, while Spur handles the coordination, review loop, and project management integration.

---

## Getting Started

Spur is delivered conveniently via an npm wrapper package. You can install it globally or run it directly using `npx`.

**Install globally via npm:**
```bash
npm install -g @getspur/spur-cli
```

**Or run directly via npx:**
```bash
npx @getspur/spur-cli tui
```

### Initialization

Spur keeps its configuration and session data isolated per repository in a `.spur` directory. Navigate to the root of your project repository in your terminal and run:

```bash
spur init
```

Start the TUI by running:
```bash
spur
```

## Documentation

Whether you are just getting started or diving deep into Spur's advanced project management features, our user guides have you covered:

1. [**Getting Started**](./docs/00-getting-started.md)
   *Prerequisites, Installation, and First Launch.*
2. [**Core Concepts & Navigation**](./docs/01-core-navigation.md)
   *Understanding the Dashboard, Compose Modes, and UI Layout.*
3. [**Session Management**](./docs/02-session-management.md)
   *Picking sessions, tracking progress, and reading agent traces.*
4. [**Commands & Input**](./docs/03-commands-and-input.md)
   *The Input Bar, Mentions (@), and the Universal Command Palette.*
5. [**Issues & Planning**](./docs/04-issues-and-planning.md)
   *The Issue Browser, Plan Inspector, and Mermaid Dependency Graphs.*
6. [**Configuration System**](./docs/05-configuration.md)
   *Understanding and customizing `.spur/config.toml` for agents.*

## Issue Tracking & Feedback

If you encounter a bug, have a feature request, or want to provide feedback, please use our GitHub Issues page:

- [Report a Bug or Request a Feature](https://github.com/getspur/spur-releases/issues)

## License

Spur is actively developed by the GetSpur team. 
