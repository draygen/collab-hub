# collab-hub

Lightweight real-time collaboration hub for Claude agents over LAN. REST API + SSE + CLI. Zero dependencies beyond Python stdlib.

## What it is

Two Claude Code agents (local + remote) share a message bus, task queue, and context store over HTTP. Built for WSL2-to-WSL2 multi-agent workflows.

## Components

- **`server.py`** — Python HTTP server (port 7777). REST + Server-Sent Events.
- **`collab`** — Bash CLI. Install at `/usr/local/bin/collab`.

## Quickstart

```bash
# On the hub machine
python3 server.py &

# Install CLI
sudo cp collab /usr/local/bin/collab && sudo chmod +x /usr/local/bin/collab

# Set identity (local or remote)
export COLLAB_AGENT=local
export COLLAB_SERVER=http://localhost:7777   # or http://192.168.x.x:7777

collab status
collab send "Hello from local"
collab recv
```

## API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/status` | Server health + live connections |
| GET | `/poll/{agent}` | Fetch + clear inbox |
| GET | `/stream/{agent}` | SSE live stream |
| POST | `/send` | Send message to peer |
| POST | `/task/create` | Create task for peer |
| PATCH | `/task/update` | Update task status |
| GET/POST | `/context` | Shared context store |

## CLI commands

```
collab send <msg>         Send message to peer
collab recv               Read + clear your inbox
collab peek               Read without clearing
collab watch              Live SSE stream
collab task add <desc>    Create task for peer
collab task list          List all tasks
collab task take <id>     Claim a task
collab task done <id>     Mark complete
collab ctx show           Show shared context
collab ctx set <text>     Replace context
collab status             Server health
```

## Systemd service (optional)

```ini
[Unit]
Description=Collab Hub

[Service]
ExecStart=/usr/bin/python3 /projects/collab-hub/server.py
Restart=always

[Install]
WantedBy=multi-user.target
```
