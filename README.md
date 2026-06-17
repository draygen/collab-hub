# collab-hub

[![Stars](https://img.shields.io/github/stars/draygen/collab-hub?style=flat-square&color=gold)](https://github.com/draygen/collab-hub/stargazers)
[![Forks](https://img.shields.io/github/forks/draygen/collab-hub?style=flat-square)](https://github.com/draygen/collab-hub/network/members)
[![License](https://img.shields.io/github/license/draygen/collab-hub?style=flat-square)](LICENSE)
[![Visitors](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fdraygen%2Fcollab-hub&count_bg=%2379C83D&title_bg=%23555555&icon=github.svg&icon_color=%23E7E7E7&title=visitors&edge_flat=true)](https://hits.seeyoufarm.com)
[![Python](https://img.shields.io/badge/python-3.8%2B-blue?style=flat-square)](https://python.org)

**Real-time collaboration hub for Claude AI agents over LAN.**  
Two agents. One message bus. Zero dependencies beyond Python stdlib.

---

## What is this?

I built this while running two Claude Code agents simultaneously — one on my laptop, one on my desktop across the room. They needed to coordinate: split tasks, share context, send files, trigger actions on each other's machines.

`collab-hub` is the result: a tiny Python HTTP server that gives agents a shared inbox, task queue, and context store. A bash CLI makes it feel native. The whole thing is ~290 lines.

```
[Laptop Agent]  ←──REST/SSE──→  [collab-hub :7777]  ←──REST/SSE──→  [Desktop Agent]
```

## Features

- **Message bus** — agents send/receive messages with delivery receipts
- **Task queue** — create, claim, and complete tasks across agents  
- **Shared context** — append/replace a shared scratchpad both agents can read
- **Live SSE stream** — `collab watch` streams incoming messages in real time
- **Zero deps** — pure Python stdlib, runs anywhere Python 3.8+ exists
- **Bash CLI** — ergonomic `collab send/recv/task/ctx` commands

## Quick Start

```bash
# 1. Start the hub (pick one machine to host it)
python3 server.py &

# 2. Install the CLI on both machines
sudo cp collab /usr/local/bin/collab && sudo chmod +x /usr/local/bin/collab

# 3. Configure each agent's identity
# On the hub machine:
export COLLAB_AGENT=local
export COLLAB_SERVER=http://localhost:7777

# On the remote machine:
export COLLAB_AGENT=remote
export COLLAB_SERVER=http://192.168.x.x:7777

# 4. Start collaborating
collab send "Hey, I just finished the auth module. Your turn on the API."
collab recv
collab task add "Write unit tests for auth module"
collab status
```

## CLI Reference

```
collab send <msg>           Send message to your peer agent
collab recv                 Read + clear your inbox
collab peek                 Read without clearing
collab watch                Live SSE stream of incoming messages

collab task add <desc>      Create a task for your peer
collab task list [status]   List tasks (pending|in_progress|done)
collab task take <id>       Claim a task
collab task done <id>       Mark complete

collab ctx show             View shared context
collab ctx set <text>       Replace shared context
collab ctx add <text>       Append to shared context

collab status               Server health + live connections
collab whoami               Show your identity
```

## REST API

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/status` | Server health, live connections, task counts |
| `GET` | `/poll/{agent}` | Fetch + clear inbox |
| `GET` | `/stream/{agent}` | SSE live stream |
| `POST` | `/send` | `{to, from, content, type}` |
| `POST` | `/task/create` | `{description, from, priority, assigned_to}` |
| `PATCH` | `/task/update` | `{id, status, ...}` |
| `GET/POST` | `/context` | Shared scratchpad |
| `GET` | `/log` | Activity log (last N entries) |

## Real-world use case

This was built live during a two-agent coding session:

- **Local agent** (desktop): coordinated Supabase setup, found API keys in Windows Credential Manager, managed GitHub pushes
- **Remote agent** (laptop): built a full Next.js rehab app, pulled files via SCP, sent Windows desktop notifications via `schtasks /IT`

Both agents used `collab send/recv` to hand off context and divide work without stepping on each other.

## Run as a service (systemd)

```ini
[Unit]
Description=Collab Hub
After=network.target

[Service]
ExecStart=/usr/bin/python3 /path/to/server.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now collab-hub
```

## Benchmarks

Tested on LAN (localhost loopback):

| Metric | Result |
|--------|--------|
| Message throughput | ~4,400 msg/run (benchmark suite) |
| SSE latency | <5ms |
| Concurrent agents | Tested with 3 |
| Server memory | ~15MB RSS |

---

Built with Python stdlib. No frameworks, no packages, no fuss.
