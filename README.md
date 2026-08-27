# APISelf MCP

Eighteen local desktop applications, served to your AI agent as MCP tools from a
single endpoint on your own machine.

Most MCP servers wrap a cloud API. These wrap applications that do things on the
machine itself: Whisper transcription, screen recording, image and video
generation, document conversion, peer-to-peer file transfer, notifications.

---

## Try it without installing anything

There is a hosted, read-only server that lets an agent browse the catalogue
before you install a thing. Four tools, no account, no local setup:

```bash
claude mcp add --transport http apiself-catalogue https://apiself.com/mcp
```

| Tool | What it does |
|---|---|
| `list_boxes` | Every box with tagline, category and maturity |
| `get_box` | One box in full: what it does, price per tier, requirements |
| `search_boxes` | Find boxes by what they do, e.g. "transcribe speech", "QR codes" |
| `get_install_command` | The exact install command for a given platform |

That is the shop window. The rest of this page is the thing itself.

---

## The local endpoint

Install the APISelf Manager, install the boxes you want, and every one of them
appears as MCP tools on:

```
http://127.0.0.1:7474/api/mcp
```

Install another box and its tools show up there with no configuration. Remove it
and they disappear. There is no per-box MCP code to write or keep in sync.

### Installing the Manager

```bash
# Linux
curl -fsSL https://apiself.com/install/en/setup.sh | bash

# macOS
curl -fsSL https://apiself.com/install/en/setup-macos.sh | bash

# Windows (PowerShell)
irm https://apiself.com/install/en/install-windows.ps1 | iex
```

The Manager runs on `localhost:7474`. Settings → AI integration prints the
ready-made configuration for each client below, with the session token filled
in.

### Connecting a client

**Claude Code** — one command, no config file:

```bash
claude mcp add --transport http apiself http://127.0.0.1:7474/api/mcp \
  --header "X-APISelf-Token: <MANAGER_SESSION_TOKEN>"
```

**Cursor** — `mcp.json`:

```json
{
  "mcpServers": {
    "apiself": {
      "url": "http://127.0.0.1:7474/api/mcp",
      "headers": { "X-APISelf-Token": "<MANAGER_SESSION_TOKEN>" }
    }
  }
}
```

**Claude Desktop** — via `mcp-remote`:

```json
{
  "mcpServers": {
    "apiself": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote", "http://127.0.0.1:7474/api/mcp",
        "--header", "X-APISelf-Token: <MANAGER_SESSION_TOKEN>"
      ]
    }
  }
}
```

Gemini CLI, GitHub Copilot and ChatGPT are also generated for you in the same
place.

---

## What you get

A full install is around 500 operations. Per box, counting only domain
endpoints (health, info and audit excluded):

| Box | Tools | What it is |
|---|---|---|
| AI Gateway | 46 | One place for AI models and keys, local and cloud |
| Auth | 37 | Self-hosted SSO and accounts for every box |
| Notify | 35 | Multi-channel notifications: email, Slack, Telegram, bring your own key |
| Video Tools | 35 | Convert, resize, trim and optimise video locally |
| Image Gen | 34 | Images from a prompt, on your GPU or via your own cloud key |
| Image Tools | 34 | Convert, resize and optimise images locally |
| S3 Bridge | 29 | A local S3 API in front of any storage backend |
| Text to Speech | 28 | Local voice synthesis, no per-character fees |
| Video Gen | 28 | AI video from a prompt via your own cloud key |
| Storage | 25 | Storage gateway: S3, SFTP, Dropbox, WebDAV |
| Recorder | 23 | Screen and microphone recording |
| AI Chat | 18 | Chat with local and cloud models over your own documents (RAG) |
| Forms | 17 | Self-hosted form builder |
| Scan Codes | 17 | QR and barcode scanning and generation, offline batch |
| Reel Forge | 16 | Prompt to finished short video: script, voice, captions |
| Doc Tools | 14 | Document conversion and PDF editing |
| Transcribe | 14 | Local Whisper transcription, fully offline |
| File Drop | 11 | Peer-to-peer file sharing, no upload to any server |

The number on your machine depends on which boxes you installed.

---

## The tool-count problem

Several hundred tools drowns an agent. Context fills with definitions the task
will never use, and selection accuracy drops long before the context window
does.

Above a threshold the server stops returning the full catalogue from
`tools/list` and returns three meta-tools instead:

| Tool | What it does |
|---|---|
| `search_tools` | Find tools by what you want to do |
| `call_tool` | Invoke one by name |
| `list_boxes` | What is installed and what each box is for |

The agent searches for what a task needs and pulls in only that. The Manager's
own tools stay directly available in both modes, because a request like
"install the QR code box" should not require a search step first.

The threshold is 50 by default and is configurable:

```bash
APISELF_MCP_TOOL_SEARCH_THRESHOLD=120   # raise it
APISELF_MCP_TOOL_SEARCH_THRESHOLD=0     # always return the full list
APISELF_MCP_TOOL_SEARCH_THRESHOLD=-1    # always search
```

---

## How the tools are generated

Every box ships an OpenAPI document describing its REST API. The Manager reads
those documents at startup and derives the MCP tool definitions from them, so:

- adding an endpoint to a box adds a tool, with no MCP code anywhere
- tool descriptions come from the API descriptions and stay in sync by
  construction
- tool names are namespaced by box, e.g. `transcribe__post_api_transcribe`
- descriptions are served in the Manager's language, not only English

Boxes also call each other over the same REST APIs, so an agent asking one box
to hand work to another is using a path that already exists rather than one
built for agents.

---

## Where things run

Everything is on `127.0.0.1` on a desktop install: no LAN listener, nothing for
`nmap` to find. Installed over SSH on a Linux machine with no display, the
Manager detects that, binds `0.0.0.0` and enables LAN access so you can reach it
from another computer. Either default can be changed during first run or later
in settings.

Boxes with AI features ship a local model as the default. Cloud models are
bring-your-own-key: the key goes into the AI Gateway box, the only place one is
stored, and other boxes route through it without ever seeing it.

---

## Things worth knowing before you install

**It is closed source.** It is a paid product from one person and there is no
version of opening it that survives that. What is true instead: your data sits
on your own disk as plain SQLite and ordinary files, and every box's HTTP
interface is documented, so nothing is trapped if APISelf disappears.

**The apps are paid, with a free tier that does not expire.** When a trial ends
the box degrades to Free rather than stopping, and nothing you created is lost.
The Manager itself is free.

**The binaries are not code-signed yet.** Windows shows a SmartScreen warning
and macOS Gatekeeper asks for confirmation on first launch. It is a cost not yet
covered and the first thing that gets paid for.

**There is no sandbox.** A box runs as a normal process under your user account,
the same as any desktop application you install.

---

## Links

- Catalogue and documentation: <https://apiself.com>
- How it fits together: <https://apiself.com/docs>
- Machine-readable catalogue: <https://apiself.com/llms.txt>
- Hosted catalogue MCP server: `https://apiself.com/mcp`

Questions about the OpenAPI-to-MCP mapping or the tool-search approach are
welcome as issues here.
