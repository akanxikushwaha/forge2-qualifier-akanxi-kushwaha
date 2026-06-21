# Architecture

## Two-Agent System

### OpenClaw — The Hands
- **Role:** Coding agent. Receives tasks, writes code, runs shell commands, reports results.
- **Model:** `google/gemini-2.5-flash` (fallback: `groq/llama-3.1-8b-instant`)
- **Channel:** `#agent-coder` — all coding tasks assigned and reported here
- **How it works:** Runs as a gateway process (`openclaw gateway`) with Slack Socket Mode. Receives `@ForgeBot` mentions and executes tasks.

### Hermes — The Brain
- **Role:** Orchestrator. Plans, remembers context across sessions, decomposes goals into tasks, runs autonomously on a schedule.
- **Model:** `gemini-2.5-flash`
- **Channel:** `#sprint-main` — plans and decisions posted here
- **Memory:** Persistent cross-session memory (repo name, branch, decisions stored and recalled)
- **Skill:** `skills/status-report/SKILL.md` — fires automatically when asked for a status update

---

## Slack Channel Scheme

| Channel | Purpose |
|---------|---------|
| `#sprint-main` | Human posts goals → Hermes posts plans → decisions made here |
| `#agent-coder` | Hermes assigns coding tasks → OpenClaw works and reports here |
| `#agent-log` | Raw agent activity, autonomous run output, audit trail |

---

## Model Routing Rationale

- **Hermes uses Gemini 2.5 Flash** — large context window needed for holding the full plan, memory, and task history simultaneously
- **OpenClaw uses Gemini 2.5 Flash / Groq llama-3.1-8b-instant** — coding tasks need fast turnaround; Groq fallback handles rate limits gracefully

---

## The Loop

```
Human posts goal in #sprint-main
        ↓
Hermes posts plan + task breakdown
        ↓
Human approves
        ↓
Hermes assigns task to OpenClaw in #agent-coder
        ↓
OpenClaw writes code, runs it, reports:
  "What I Did / What's Left / What Needs Your Call"
        ↓
Human reviews and approves or corrects
        ↓
Loop repeats for next task
```

Everything is visible in Slack. No agent works in private.
