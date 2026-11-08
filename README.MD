# Mini ClawDBot

> A CLI-first AI coding agent with three distinct modes — **Ask**, **Plan**, **Write** — plus a Telegram bot interface. Built with Bun, TypeScript, and the Vercel AI SDK.

## ✨ Features

### Three CLI Modes

| Mode | Purpose | Capabilities |
|------|---------|--------------|
| **Ask** 🤔 | Read-only Q&A | Search codebase, read files, analyze structure, web search — **never writes** |
| **Plan** 🧭 | Task decomposition | Generates a step-by-step plan, lets you select steps, then executes them with full tool access |
| **Write** 🤖 (Agent) | Autonomous coding | Full read/write/execute access with **staged changes + approval flow** before anything hits disk |

### Telegram Bot 📱
Run the same agent capabilities through a Telegram bot — great for mobile or remote access.

### Safety First
- **Staged mutations**: All file writes are staged in memory until you explicitly approve
- **Diff preview**: See exactly what changes will be applied
- **Granular permissions**: Each mode has different default permissions (Ask = read-only, Plan/Agent = full)
- **Shell execution opt-in**: Commands require explicit configuration

---

## 🚀 Quick Start (Local Development)

### Prerequisites
- **Bun** ≥ 1.1.0 — `curl -fsSL https://bun.sh/install | bash`
- **OpenRouter API key** — Get one at [openrouter.ai](https://openrouter.ai/keys)

### 1. Clone & Install
```bash
git clone https://github.com/yourusername/mini-clawdbot.git
cd mini-clawdbot
bun install
```

### 2. Configure Environment
Create `.env` in the project root:
```bash
cp .env.example .env  # if exists, or create manually
```

Edit `.env` with your keys:
```env
# Required
OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxxx

# Optional: for Telegram bot
TELEGRAM_BOT_TOKEN=123456:ABC-DEF...
TELEGRAM_OWNER_ID=123456789
```

### 3. Run
```bash
# Interactive menu (choose CLI or Telegram)
bun run index.ts wakeup
```

---

## 🎮 Usage

### CLI Modes (via `bun run index.ts wakeup` → select "CLI")

| Mode | Command | Use Case |
|------|---------|----------|
| **Ask** | Select "Ask" | Read-only questions, codebase analysis, web search |
| **Plan** | Select "Plan" | Break down a goal → review steps → execute selected ones |
| **Agent** | Select "Agent" | Full autonomous coding with approval before writes |

#### Example Flows

**Ask Mode**
```
❓ Ask Mode
What do you want to ask? > How does the approval flow work?
```
- Reads files, searches codebase, analyzes structure
- Can optionally save answer as `.md` (with approval)
- **Never modifies your codebase**

**Plan Mode**
```
🧭 Plan Mode
What is your goal? > Add a new CLI command for exporting conversation history
```
1. Agent generates a **step-by-step plan**
2. You **select which steps to execute**
3. Each step runs with full tool access
4. Changes staged → you approve → applied

**Agent (Write) Mode**
```
🤖 Agent Mode
What would you like the agent to do? > Refactor the auth module to use JWT
```
- Autonomous loop (up to 40 steps)
- Full tool access: read, write, edit, delete, shell, search, web
- **All mutations staged** — you review a diff before anything is written

### Telegram Mode
```bash
# Ensure TELEGRAM_BOT_TOKEN and TELEGRAM_OWNER_ID are set in .env
bun run index.ts wakeup
# → Select "Telegram"
```
Bot sends a welcome message to `TELEGRAM_OWNER_ID` and starts polling. Use `/ask`, `/plan`, `/agent` commands in chat.

---

## ⚙️ Configuration

### Environment Variables
| Variable | Required | Description |
|----------|----------|-------------|
| `OPENROUTER_API_KEY` | Yes | API key for OpenRouter (or compatible) |
| `TELEGRAM_BOT_TOKEN` | No | Bot token from @BotFather |
| `TELEGRAM_OWNER_ID` | No | Your Telegram user ID (numeric) |

### Model Configuration
Edit `ai/ai.config.ts` to change the default model:
```ts
export function getAgentModel() {
  return openrouter('anthropic/claude-3.5-sonnet'); // or any OpenRouter model
}
```

### Tool Permissions
Default permissions per mode (in `modes/agent/types.ts`):
```ts
export const defaultAgentConfig = () => ({
  codebasePath: process.cwd(),
  tools: {
    allowFileCreation: true,
    allowFileModification: true,
    allowFolderCreation: true,
    allowShellExecution: false,  // opt-in only
  },
});
```

---

## 📁 Project Structure

```
mini-clawdbot/
├── index.ts                 # CLI entry point (commander)
├── package.json
├── tsconfig.json
├── .env                     # Your secrets (not committed)
│
├── ai/                      # AI provider configuration
│   ├── ai.config.ts         # Model selection (OpenRouter)
│   └── index.ts
│
├── modes/
│   ├── cli.ts               # CLI sub-mode selector
│   │
│   ├── agent/               # WRITE MODE (full autonomy)
│   │   ├── orchestrator.ts  # Main agent loop
│   │   ├── agent-tools.ts   # Tool definitions (read/write/edit/shell/...)
│   │   ├── tool-executor.ts # Executes & stages mutations
│   │   ├── action-tracker.ts# Tracks pending changes
│   │   ├── approval.ts      # Interactive diff approval UI
│   │   ├── diff-view.ts     # Pretty diff rendering
│   │   └── types.ts         # Config & types
│   │
│   ├── ask/                 # ASK MODE (read-only)
│   │   └── orchestrator.ts  # Q&A agent with web search
│   │
│   ├── plan/                # PLAN MODE (plan → select → execute)
│   │   ├── orchestrator.ts  # Plan flow
│   │   ├── planner.ts       # Generates plan from goal
│   │   ├── selection.ts     # Interactive step picker
│   │   ├── web-tools.ts     # Web search / fetch tools
│   │   └── types.ts         # Plan step types
│   │
│   └── telegram/            # Telegram bot interface
│       ├── index.ts         # Bot startup
│       ├── handlers.ts      # Command handlers (/ask, /plan, /agent)
│       ├── auth.ts          # Owner-only protection
│       ├── approval-session.ts
│       ├── plan-session.ts
│       ├── constants.ts
│       └── text.ts
│
└── tui/
    ├── terminal-md.ts       # Markdown → terminal renderer
    └── wakeup.ts            # Banner + mode selector
```

---

## 🔧 Available Tools

| Tool | Ask | Plan | Agent | Description |
|------|-----|------|-------|-------------|
| `read_file` | ✅ | ✅ | ✅ | Read any text file |
| `list_files` | ✅ | ✅ | ✅ | List directory contents |
| `search_files` | ✅ | ✅ | ✅ | Glob + content search |
| `analyze_codebase` | ✅ | ✅ | ✅ | Project summary (counts, size, ext) |
| `list_skills` / `read_skill` | ✅ | ✅ | ✅ | Browse SKILL.md files |
| `create_file` | ❌ | ✅ | ✅ | Create new files |
| `edit_file` | ❌ | ✅ | ✅ | Modify existing files |
| `delete_file` | ❌ | ✅ | ✅ | Delete files |
| `create_folder` | ❌ | ✅ | ✅ | Make directories |
| `execute_shell` | ❌ | ⚠️ | ⚠️ | Run shell commands (opt-in) |
| `web_search` / `web_fetch` | ✅ | ✅ | ✅ | Web search via Firecrawl |

> **Note**: In Ask mode, `allowFileCreation` is enabled only for saving answers as `.md` files.

---

## 🛡️ Approval Flow

All write operations (create, edit, delete, shell) are **staged in memory**. Before anything touches your filesystem:

1. Agent finishes its work
2. You see a **summary of pending changes**
3. Interactive prompt: **Approve / Reject / View Diff**
4. If approved → changes applied atomically
5. If rejected → staging cleared, nothing written

```bash
# Example approval prompt
? Pending changes (3):
  ✏️  M  src/auth.ts
  ✏️  M  src/utils.ts
  ➕  A  tests/auth.test.ts
  
  [A]pprove all  [R]eject all  [D]iff  [S]elect individually
```

---

## 🛠️ Development

### Type Check
```bash
bunx tsc --noEmit
```

### Run Tests (if any)
```bash
bun test
```

---

## 🤝 Contributing

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/amazing-feature`
3. Make your changes
4. Run typecheck: `bunx tsc --noEmit`
5. Submit a PR

---

## 📄 License

MIT — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgements

- [Vercel AI SDK](https://sdk.vercel.ai/docs) — agent loop & tool calling
- [Bun](https://bun.sh/) — fast runtime & bundler
- [@clack/prompts](https://github.com/clack/prompts) — beautiful terminal UIs
- [OpenRouter](https://openrouter.ai/) — unified LLM API
- [Firecrawl](https://firecrawl.dev/) — web search/scraping
- [Telegraf](https://telegraf.js.org/) — Telegram bot framework