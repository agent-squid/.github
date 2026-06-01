# 🦑 Agent-Squid
```
                   🦑 AGENT
    ██████╗ ██████╗ ██╗   ██╗██╗██████╗
   ██╔════╝██╔═══██╗██║   ██║██║██╔══██╗
   ╚█████╗ ██║   ██║██║   ██║██║██║  ██║
    ╚═══██╗██║▄▄ ██║██║   ██║██║██║  ██║
   ██████╔╝╚██████╔╝╚██████╔╝██║██████╔╝
   ╚═════╝  ╚══▀▀═╝  ╚═════╝ ╚═╝╚═════╝
```

**A local web cockpit for CLI coding agents.**

Agent-Squid lets you run Claude Code, OpenAI Codex, Cursor Agent, GitHub Copilot CLI, Google Antigravity, and other local agent CLIs from one browser UI. Your agents still run on your own machine, with your repo, shell tools, credentials, and native CLI sessions. Squid gives those sessions names, history, queues, controls, and phone/tablet access.

Use it when your workflow has become a wall of unnamed terminal tabs.

```text
#launch@claude write the release notes
#launch@codex! review the diff for regressions
#bug@cursor reproduce the auth failure
#ops@copilot summarize the incident notes
```

## Why

CLI agents are useful, but the workflow gets messy quickly:

- You have ten Claude Code terminals open and cannot remember which one is doing what.
- A terminal closes or the machine reboots, and reviving the right session becomes a hunt through resume pickers, session IDs, cwd-sensitive history, and shell scrollback.
- Long-running sessions grow, drift, stall, and need compaction or reset.
- It is hard to compare a fully resumed session against a fresh prompt with only limited historical context.
- Different agent CLIs have different commands, resume flags, slash commands, output formats, and UI assumptions.
- Token usage is usually hidden in the flow instead of attached clearly to each prompt and response.
- The best machine for the job is often not the device in your hand. It is your Mac mini, workstation, or always-on local box with the repo and CLIs already configured.

Squid turns those local agents into named, durable lanes you can control from a browser.

Topics and agents are not a rigid setup step. You can create a new `#topic` the moment you type it, switch agents with `@agent`, and clean up old workstreams when they are done. Squid treats tags as the UI for your agent work: lightweight enough to create dynamically, durable enough to recover later, and visible enough that you are not guessing which terminal was doing what.

## What You Get

- **Named agent lanes:** `#topic@agent` gives every workstream a durable identity.
- **Dynamic tags:** create topics as you type, reuse sticky agents, filter by tag, and delete stale topics when the work is done.
- **Native resumable sessions:** Squid tracks session handles while the CLI owns its real context.
- **Parallel work:** different topics and agents run independently.
- **Adhoc mode:** `#topic@agent!` runs a fresh one-off job immediately without polluting the main session.
- **Session vs. limited-context comparison:** compare a fully resumed lane against an adhoc prompt that includes only the last N exchanges.
- **Live progress bubble:** watch queued state, tool/status output, and partial response progress while the CLI is working.
- **Auto-compaction settings:** keep long-running lanes useful without manually babysitting context forever.
- **Context bookmarks:** pin a useful answer and inject it into another session or adhoc turn.
- **Process controls:** stop one process from the UI, stop by command, stop a topic, drain queues, clear sessions, and compact/reset context.
- **History and filtering:** scan past work by topic, agent, or adhoc lane.
- **Analytics:** review usage by time, topic, or agent, plus live process state.
- **Per-prompt usage:** every completed prompt can show input, output, cache, reasoning, cost, duration, and quota signals when the backend exposes them.
- **Phone/tablet access:** lie on the couch while your local machine keeps coding.

## Quick Start

```bash
git clone https://github.com/agent-squid/squid
cd squid
cp config/squid.yaml.example config/squid.yaml
bin/install.sh
bin/start.sh
```

Open in your browser:

```text
http://127.0.0.1:8000
```

Install at least one supported CLI:

| Backend | CLI | Install |
|---|---|---|
| Claude Code | `claude` | `npm install -g @anthropic-ai/claude-code` |
| OpenAI Codex | `codex` | `npm install -g @openai/codex` |
| Cursor Agent | `cursor-agent` | install from Cursor |
| GitHub Copilot | `copilot` | `gh extension install github/gh-copilot` |
| Google Antigravity | `agy` | install from Antigravity |

Create agents in the UI. An agent is a named config:

```text
name + backend + model + working directory
```

Examples:

```text
claude-main  -> claude, default model, /tmp/squid/work
codex-review -> codex,  default model, /tmp/squid/review
```

## Basic Usage

| Syntax | Meaning |
|---|---|
| `#topic@agent message` | Continue that topic/agent session |
| `#topic message` | Use the sticky agent for that topic |
| `#topic@agent! message` | Run a parallel adhoc turn with no session history |
| `#topic@agent!3 message` | Run adhoc with only the last 3 exchanges as context |
| `/stop` | Stop the current process scope |
| `/stopall` | Stop and drain the current topic |
| `/clear` | Clear the current session |
| `/compact` | Compact or reset context |
| `/filter` | Filter history to the current topic/agent lane |
| `/remote` | Show QR code for mobile/tablet access via Tailscale |

Session turns are queued per `#topic@agent` because order matters. Adhoc `!` turns are independent and run immediately.

This makes comparison easy. Ask the fully resumed session with `#topic@agent`, then ask a limited-context version with `#topic@agent!3`, `#topic@agent!1`, or `#topic@agent!`. You can see whether the long session is helping, whether stale context is hurting, and how much token usage each path costs.

Tags are created dynamically. If you send `#release@claude ...`, Squid creates or updates the `release` topic and remembers `claude` as its sticky agent. Later, `#release ...` continues that lane without retyping the agent. Topic autocomplete lets you browse existing tags, and old topics can be hidden or deleted when they are no longer useful.

While an agent is running, Squid shows progress in a thought bubble instead of leaving you staring at a frozen terminal. You can see queue position, status output, tool activity, and streamed partial content. If the job is going in the wrong direction, stop it from the bubble’s UI control or type `/stop`, `/stopall`, or `deq` just like you would use CLI control commands, but with clearer scope and feedback.

After completion, the response carries usage metadata. Instead of guessing what a turn cost, you can inspect tokens, cache reads/writes, reasoning tokens where available, duration, cost, and quota deltas. The analytics view rolls this up by time, topic, and agent so you can see which lanes are expensive and which models are doing the most work.

## Couch Coding With Tailscale

Squid is most useful when your local machine can keep working while you are away from the desk.

Tailscale is a good fit for this. Its Personal plan is free for non-commercial personal use, and it creates a private WireGuard-based network across your own devices. Your phone, tablet, laptop, Mac mini, and workstation can talk inside the tailnet without opening a public port.

Squid always binds to `127.0.0.1` — it never exposes itself directly on the network. `bin/start.sh` automatically configures Tailscale’s HTTPS proxy if Tailscale is installed:

```bash
bin/start.sh   # configures tailscale serve automatically
```

Access from any enrolled device at:

```text
https://<machine-name>/
```

Tailscale handles TLS — browsers show the padlock, no port number needed. On first visit from a new device, add your token:

```text
https://<machine-name>/?token=<your-token>
```

Or type `/remote` in the chat on your laptop to get a QR code — point your phone camera at it to open squid authenticated in one tap.

## How Squid Is Different

Squid is not another general AI chat app. Open WebUI and LibreChat are broad self-hosted chat platforms for many providers, RAG, plugins, users, memory, and web search.

Squid is narrower: it controls real local coding-agent CLIs and preserves their session behavior.

Most chat UIs send messages to a model API and render text back. Even when they support tools or agents, the chat app is usually the runtime. Squid is different: the runtime is still the local CLI agent. Claude Code, Codex, Cursor Agent, Copilot CLI, or Antigravity is the process doing the work on your machine. Squid is the interactive control layer around those processes.

That difference matters:

- **Real CLI sessions, not copied chat history:** Squid resumes the agent’s native session instead of pretending to be the agent with a replayed transcript.
- **Working-directory awareness:** sessions are tied to the cwd where the CLI actually runs, which matters for local project context and resume behavior.
- **Process ownership:** Squid can show live processes, kill the exact running job, drain queued jobs, and recover after disconnects.
- **Topic and agent lanes:** `#topic@agent` is closer to a named terminal workspace than a chat room.
- **Progress while work happens:** the thought bubble surfaces status, queued state, tool activity, and partial output before the final answer lands.
- **UI plus command control:** click to stop a specific run, or type commands like `/stop`, `/stopall`, `/clear`, `/compact`, and `/filter`.
- **Analytics attached to real work:** token usage is tied to each prompt and can be rolled up by topic, agent, or time.
- **Built-in context experiments:** compare a native resumed session against an adhoc turn with only selected recent context.
- **Local machine as the backend:** your Mac mini or workstation stays the execution environment, so the agent has the same filesystem, shell, credentials, and installed tools it would have in a terminal.

| Category | Examples | Squid's difference |
|---|---|---|
| Self-hosted AI chat | Open WebUI, LibreChat | Squid runs local CLI coding agents instead of replacing them with a provider chat UI. |
| Single-agent CLIs | Claude Code, Codex CLI, Cursor Agent, Copilot CLI | Squid gives them shared browser/mobile UI, topics, queues, history, controls, and analytics. |
| IDE agents | Cursor, Cline, VS Code Copilot | Squid is editor-agnostic and works even when the IDE is not open. |
| Terminal pair programmers | Aider, OpenCode | Squid is an orchestration layer, not a coding engine. |

Use the agent directly when one terminal is enough. Use Squid when you want several local agents, named sessions, mobile access, process control, recoverable long-running work, and analytics that explain what each lane is costing you.

## Architecture

```text
Browser / phone / tablet
        |
        | HTTP + SSE
        v
FastAPI Squid server
        |
        +-- SQLite history, stats, topics, session handles
        |
        +-- TopicDispatcher
              |
              +-- FIFO worker per #topic@agent session lane
              +-- ephemeral worker per adhoc ! turn
        |
        +-- local CLI subprocesses
              claude / codex / cursor-agent / copilot / agy
```

The CLI owns the real conversation context. Squid stores history, stats, topics, active session IDs, cwd locks, and UI state needed to make the workflow manageable. Session turns use native resume. Adhoc turns build an explicit limited-context prompt, which makes the two modes easy to compare.

## Good Fit

Squid is a good fit if:

- You run multiple coding-agent terminals at once.
- You want to recover and operate sessions by name.
- You use a local Mac mini, workstation, or always-on machine for agent work.
- You want phone/tablet control without moving execution to the cloud.
- You compare or combine several agent CLIs.
- You want to compare full-session context against limited-history prompts.
- You care about token usage, cost, cache behavior, and per-topic or per-agent analytics.

It may be overkill if:

- You only ever use one agent in one terminal.
- You want a hosted team chat product.
- You want Squid to be the coding agent itself.
