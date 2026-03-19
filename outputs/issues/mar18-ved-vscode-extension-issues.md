# Issues for Ved — VS Code Extension Capture Layer
**Created:** March 18, 2026
**Source:** Mar 18 product meeting (Timothy + Hitesh + Ved)
**Assignee:** Ved

---

## Context

Ved proposed embedding the Sploink capture layer directly inside the VS Code extension. Instead of requiring the user to run a separate CLI (`sploink run -- claude "task"`), the extension becomes the interface: user types prompt into the extension panel, picks which agent to call, extension captures all events in real time. CLI remains available for terminal-first users, but the extension becomes the primary surface.

This doesn't replace SAL-36 → SAL-31 → SPO-85. Those still go first. These issues describe the architecture that follows once the core loop is working.

---

## SPO-86 — VS Code Extension: Unified Agent Interface

**Priority:** High (after core loop is live)
**Blocks:** SPO-87, SPO-88

**What:**
Embed the Sploink capture layer directly inside the VS Code extension as an AI assistant panel — similar to the GitHub Copilot Chat sidebar or the Cursor agent panel. The user types their prompt into the panel, picks which agent handles it, and the extension runs the session. Sploink capture runs silently in the background — no separate terminal window needed.

**Why:**
The current flow (open terminal, type `sploink run -- claude "task"`) adds friction. Developers already live in VS Code. An extension panel removes the context switch entirely. Also enables a tighter feedback loop: the IDE is the place where bugs are visible, so surfacing drift signals there (SPO-85) makes most sense when the session also starts there.

**Acceptance criteria:**
- Extension panel renders an input field for the user's prompt
- User can submit a prompt and have it routed to the selected agent
- Session capture starts automatically when prompt is submitted — no manual `sploink run` required
- Telemetry emits to Salaxia on session completion, same as CLI flow
- CLI still works independently — extension is additive, not a replacement

**Dependencies:** SAL-36 (Salaxia deployed), SPO-85 (IDE highlighting live)

---

## SPO-87 — Extension: Agent Selection Panel

**Priority:** High (same phase as SPO-86)
**Depends on:** SPO-86

**What:**
Inside the extension panel, a dropdown or list showing available agents the user can route their prompt to — Claude, Cursor, Codex, or any agent installed on the machine. User selects the agent before submitting the prompt. The selected agent is recorded as a telemetry event in the session.

The first version should default to whatever agent is installed (probably Claude Code). Multi-agent switching can come later — just make the field present and wired.

**Why:**
Agent selection is the seed of the routing layer. Even if users only use one agent today, capturing which agent was used per session is how we build agent performance data over time. This field is required before SAL-32 (unified session score feeding agent rankings) can be meaningful.

**What to build:**
- Dropdown in the panel showing detected agents (check for `claude`, `cursor`, `codex` binaries on PATH)
- Selected agent captured as a `session_meta` event with field `agent_name`
- Default: first detected agent. Falls back gracefully if none detected.

**Acceptance criteria:**
- Dropdown renders with at least one agent (or a clear empty state if none found)
- Selected agent is emitted as a telemetry field on the session
- Selection persists across sessions (remembers last choice)

**Dependencies:** SPO-86

---

## SPO-88 — Extension: Real-time Event Capture Without CLI Wrapper

**Priority:** High (same phase as SPO-86)
**Depends on:** SPO-86

**What:**
When a session runs through the extension panel (not the CLI), capture all events directly from the extension — prompt submitted, files edited, agent responses received, tool calls made. These events should conform to the same canonical schema (SPO-21) as CLI-captured events so SAL-31 and all downstream scoring work identically regardless of how the session was started.

**Why:**
The CLI wrapper works by spawning the agent as a subprocess and attaching watchers. The extension can't do that the same way — it's already inside the IDE and has direct access to the file system, editor state, and agent API calls. This is actually a richer capture surface. But it needs to emit the same event types so nothing downstream has to change.

**What to capture from the extension:**
- `prompt_submitted` — user's input text, agent selected, timestamp
- `file_edit` — VS Code `onDidChangeTextDocument` API, same fields as CLI version
- `agent_response` — response received from agent, token count, latency
- `tool_call` — if agent calls a tool (file read, terminal run), capture it
- `shell_command` — if agent runs terminal commands via VS Code terminal, capture exit code + output

**Acceptance criteria:**
- All events conform to SPO-21 schema — no new fields added, no existing fields missing
- Session from extension produces a telemetry payload identical in structure to a CLI session
- SAL-31 drift scorer receives and scores extension-captured sessions without modification
- SPO-45 (data fusion) can merge CLI + extension events into one timeline when both are running

**Dependencies:** SPO-86, SAL-31

---

## Sequencing

These three issues are a single unit. Build SPO-86 first (the shell), then SPO-87 and SPO-88 in parallel since they're independent wiring tasks inside SPO-86.

Don't start these until SAL-36 and SAL-31 are done. The extension needs a live Salaxia endpoint to emit to, and the capture layer isn't useful without drift scoring downstream.

**Full order:**
```
SAL-36 (deploy Salaxia)
  → SAL-31 (drift from error signal)
    → SPO-85 (IDE highlighting)
      → SAL-29 / SAL-26 (intervention prompt)
        → SPO-86 / SPO-87 / SPO-88 (extension capture — this batch)
```
