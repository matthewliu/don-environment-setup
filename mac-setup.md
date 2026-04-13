# Matt's New Mac Setup

Provisioning a fresh Mac (Apple Silicon) for coding with Claude Code, Cursor, and the claude-toolkit.
Mirrors the existing environment from the current machine.

---

## Priority Order

### Tier 1: Code TODAY
- [Xcode CLI + Homebrew](#1-xcode-cli--homebrew)
- [Git + GitHub](#2-git--github)
- [Node.js + global packages](#3-nodejs--global-packages)
- [Claude Code + Codex](#4-claude-code--codex)
- [Cursor IDE](#5-cursor-ide)

### Tier 2: Full dev environment
- [Python 3.12](#6-python)
- [PostgreSQL 16 + Redis + SQLite](#7-databases)
- [Deployment CLIs](#8-deployment-clis)
- [Utility CLIs](#9-utility-clis)

### Tier 3: Toolkit + specialized
- [Claude Toolkit deploy](#10-claude-toolkit)
- [Security tools](#11-security-tools)
- [Docker](#12-docker)

### Tier 4: When needed
- [Rust (Joyride only)](#13-rust)
- [Ruby + CocoaPods (iOS only)](#14-ruby--ios)
- [Mobile dev (Expo, Android Studio, Xcode)](#15-mobile-dev)

---

## Tier 1: Code TODAY

### 1. Xcode CLI + Homebrew

```bash
# Xcode Command Line Tools (required for almost everything)
xcode-select --install

# Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add to PATH (Apple Silicon)
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### 2. Git + GitHub

```bash
brew install git gh

# Configure
git config --global user.name "Matthew Liu"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
git config --global pull.rebase false

# Authenticate
gh auth login
```

### 3. Node.js + Global Packages

```bash
# Node.js (current: v25, or use LTS v22)
brew install node

# Global packages (matching current machine)
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
npm install -g @google/gemini-cli
npm install -g pnpm
npm install -g bun
npm install -g tsx
npm install -g concurrently
npm install -g vercel
npm install -g eas-cli
npm install -g @tobilu/qmd
```

### 4. Claude Code + Codex

```bash
# Already installed above via npm, verify:
claude --version
codex --version

# Add API keys to ~/.zshrc
cat >> ~/.zshrc << 'KEYS'

# AI API Keys
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export GOOGLE_API_KEY="AI..."
export XAI_API_KEY="xai-..."
KEYS

source ~/.zshrc
```

### 5. Cursor IDE

```bash
brew install --cask cursor
```

Extensions to install:
- ESLint, Prettier, Tailwind CSS IntelliSense
- Python (Microsoft), GitLens, Error Lens

---

## Tier 2: Full Dev Environment

### 6. Python

```bash
brew install python@3.12 pipx virtualenv

# Add pipx to PATH
pipx ensurepath

# Global Python CLI tools
pipx install copier
pipx install ruff
```

### 7. Databases

```bash
# PostgreSQL 16 (primary DB for all projects)
brew install postgresql@16
brew services start postgresql@16
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
createdb $(whoami)

# pgvector extension (vector similarity)
brew install pgvector

# Redis (for x-business-intelligence)
brew install redis
brew services start redis

# SQLite (already included with macOS)
sqlite3 --version
```

### 8. Deployment CLIs

```bash
brew install railway          # Railway (AgenticMiniApps, AgentToolkit)
```

Not needed: Heroku, AWS CLI, hcloud (install only if a specific project requires them).

### 9. Utility CLIs

```bash
brew install ripgrep          # Fast search (Claude Code uses this)
brew install jq               # JSON processing
brew install tmux             # Terminal multiplexer
brew install watch            # Repeat commands
brew install overmind         # Process manager (Procfile runner)
brew install coreutils        # GNU coreutils
brew install openssl          # TLS/crypto
```

Optional (install if needed):
```bash
brew install graphviz         # Diagram rendering
brew install ffmpeg           # Media processing
brew install cloudflared      # Cloudflare tunnels
```

---

## Tier 3: Toolkit + Specialized

### 10. Claude Toolkit

```bash
# Clone (use your actual repo URL)
cd ~/Environments/AgentToolkit
git clone https://github.com/matthewliu/claude-toolkit.git
cd claude-toolkit

# Install MCP review server deps
cd mcp-servers/review-server && npm install && cd ../..

# Deploy to ~/.claude/
./sync.sh push

# Enable plugin in ~/.claude/settings.json:
#   "enabledPlugins": { "matt-toolkit": true }

# Optional: Playwright MCP for browser automation
claude mcp add playwright -- npx @playwright/mcp@latest

# Optional: StatusLine HUD
# In ~/.claude/settings.json:
#   "statusLine": { "type": "command", "command": "node $HOME/.claude/hud/tli-hud.mjs" }
```

### 11. Security Tools

```bash
brew install gitleaks         # Secret scanning
pipx install semgrep          # Static analysis
```

### 12. Docker

```bash
brew install --cask docker
# Open Docker Desktop to finish setup
```

---

## Tier 4: When Needed

### 13. Rust

Only for Joyride backend.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

### 14. Ruby + iOS

Only for building iOS apps locally.

```bash
brew install rbenv ruby-build
rbenv install 3.2.0
rbenv global 3.2.0
gem install cocoapods
```

### 15. Mobile Dev

```bash
# Watchman (file watcher for React Native)
brew install watchman

# Android Studio
brew install --cask android-studio

# Xcode: Install from App Store, then:
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
```

---

## Cloud Accounts

### Already have (transfer/re-auth on new machine)
- GitHub, Anthropic, OpenAI, Google Cloud, xAI
- Railway, Vercel, Docker Hub
- Logto, SendGrid

### API keys to migrate

Copy from old machine's `~/.zshrc` (or password manager):

```bash
# Core (toolkit reviews + AI tools)
ANTHROPIC_API_KEY
OPENAI_API_KEY
GOOGLE_API_KEY
XAI_API_KEY

# Project-specific (add as needed)
GROQ_API_KEY
PERPLEXITY_API_KEY
SENDGRID_API_KEY
X_BEARER_TOKEN
```

### Settings to migrate

These are the critical dotfiles/configs to copy from the old machine:

```
~/.claude/settings.json       # Claude Code settings, permissions, plugins
~/.claude/CLAUDE.md           # Global instructions
~/.claude/PREFERENCES.md      # Personal preferences (Obsidian path, etc.)
~/.claude.json                # MCP server registrations
~/.gitconfig                  # Git config
~/.zshrc                      # Shell config + API keys
~/.ssh/                       # SSH keys (or generate new ones)
```

The toolkit itself doesn't need to be copied -- just clone it fresh and run `./sync.sh push`.

---

## Full One-Shot Script

If you want to run everything in one go (Tiers 1-3):

```bash
# Prerequisites: Xcode CLI tools + Homebrew already installed

# Languages & tools
brew install git gh node python@3.12 pipx virtualenv
brew install ripgrep jq tmux watch overmind coreutils openssl

# Databases
brew install postgresql@16 redis pgvector
brew services start postgresql@16
brew services start redis
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc

# Deployment
brew install railway

# Security
brew install gitleaks

# GUI apps
brew install --cask cursor docker

# Node globals
npm install -g @anthropic-ai/claude-code @openai/codex @google/gemini-cli
npm install -g pnpm bun tsx concurrently vercel eas-cli @tobilu/qmd

# Python globals
pipx install copier ruff semgrep

# Create default DB
source ~/.zshrc
createdb $(whoami)

echo "Done. Add API keys to ~/.zshrc, clone toolkit, run sync.sh push."
```

---

## Verify

```bash
echo "=== Core ==="
node --version          # v25.x
npm --version           # 11.x
python3 --version       # 3.12.x
git --version           # 2.48+
gh --version            # 2.88+

echo "=== AI ==="
claude --version        # Claude Code 2.x
codex --version         # Codex 0.x

echo "=== Databases ==="
psql --version          # 16.x
sqlite3 --version       # 3.x
redis-cli ping          # PONG

echo "=== Package Managers ==="
pnpm --version
bun --version

echo "=== Deploy ==="
railway version
vercel --version

echo "=== Search ==="
rg --version            # ripgrep
```
