# Known Issues

This page lists rough edges we are aware of and actively working on. None of these will damage your project or your work — they are mostly about **what to expect** so you are not caught off guard.

If you hit something not listed here, please open an issue on the public tracker.

---

## 1. First launch in a new project can feel slow

**What you might see:** the TUI sits on a loading screen for **30 seconds to over a minute** the first time you start Spur in a project.

**Why:** Spur boots an underlying coding agent (such as Claude Code) the first time you use a project. On a cold machine this includes downloading and resolving the agent package, then handing the conversation over once it is ready. Once the agent is warm, subsequent launches in the same project are noticeably faster.

**What to do:**
- Be patient on the very first launch — there is no progress bar yet, but it is making progress.
- After the first run, the agent is cached, so day-to-day startup is much quicker.
- If startup feels slow every time (not just the first), see issue #3 below.

---

## 2. Resuming a long-running session takes longer than starting fresh

**What you might see:** when you resume an old session — especially one with a long conversation history — the `Loading session…` step takes several seconds before the chat appears.

**Why:** Spur replays the prior conversation so the agent has full context. The longer the conversation, the more there is to replay.

**What to do:**
- For short or recent sessions you should not notice this.
- If a session is very large and you do not need its history, starting a fresh session in the same project is faster.
- We are planning a "lightweight resume" mode that streams the most recent context first.

---

## 3. Running multiple Spur windows on the same project slows everything down

**What you might see:** if you have **two or more `spur tui` sessions open against the same project directory at the same time**, you may notice:

- Slower session loads
- Slower responses to commands like `/issues` or `/plans`
- Occasional warning messages in the log about plans being owned by another brain

**Why:** Spur shares a single project state file (the issue and plan database) between windows. When several Spur windows compete for it, they have to take turns, and that adds up.

**What to do:**
- For now, **prefer one Spur window per project**. You can still switch between sessions inside that one window using the Session Picker (`Alt+s`).
- If you want two views of the same project, opening one in the TUI and the other in a read-only viewer is fine — the issue is two TUIs both trying to coordinate work.

---

## 4. Some old plans show "owned by another brain"

**What you might see:** in older projects you may notice plans that appear inactive, with log lines mentioning `PlanOwnedByAnotherBrain` or owners that look like long random IDs.

**Why:** every brain session that worked on a plan stamps its identifier on it, so two brains will not step on each other's work. Over time, plans accumulate stamps from past sessions that have since gone away. Spur is being conservative and refusing to take over those plans automatically.

**What to do:**
- Most users can ignore this — it does not block anything.
- If you want to actively continue a plan that shows as owned by an older session, use the **`Force reclaim plan`** action (also exposed as `force_reclaim_plan` for advanced users). This safely transfers ownership to your current session.

---

## 5. Repeated warnings about audit comments in the log

**What you might see:** if you tail Spur's log file, you may see warnings like:

```
audit sentinel parse failed; comment dropped from projection
```

repeating every few seconds for a small number of issues.

**Why:** Spur writes machine-readable audit comments alongside human-readable ones, and an older format is incompatible with the current parser. The parser refuses to guess, so it drops those few comments and re-warns on every refresh.

**What to do:**
- This does **not** affect your work, your issues, or your plans.
- The warnings will go away in a future release that makes the parser tolerant of the older format.
- If the noise bothers you, you can lower Spur's log level — see [Configuration](./05-configuration.md).

---

## 6. New session creation can stall behind a slow agent download

**What you might see:** creating a brand-new session — particularly with a brain you have not used before — can pause for tens of seconds before the first reply.

**Why:** the underlying agent is downloaded on demand the first time you use it. Subsequent sessions for the same brain reuse the cached install.

**What to do:**
- The pause is one-time per agent.
- If the install seems to fail (no progress for several minutes), check that you have a working internet connection and that `npx` / `npm` is installed and reachable.

---

## Reporting a new issue

If you run into something that is not on this list, the most useful information you can include is:

1. The exact thing you did just before the issue (e.g. "switched session", "resumed session XYZ", "opened a second Spur window").
2. Whether the issue is reproducible or one-off.
3. Roughly when it happened (Spur's logs are timestamped).

That makes it dramatically easier for us to track down. Thanks for trying Spur.
