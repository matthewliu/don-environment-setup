# Don's Windows 11 Dev Machine Setup

Setting up Windows 11 Home for coding with Claude Code, Cursor, and Matt's toolkit.

**Key decision: WSL2 is your primary dev environment.** Windows 11 Home supports WSL2 fully. You'll install GUI apps (Cursor, Docker Desktop) on Windows, but do all terminal work inside WSL2's Ubuntu. This gives you the same Linux environment Matt runs on Mac, so commands, scripts, and tutorials all work identically.

---

## Priority Order

This is sequenced so you're productive fast. Each tier builds on the previous.

### Tier 1: Code TODAY (do this first sitting)
- [Windows foundations + WSL2](#1-windows-foundations--wsl2)
- [Git + GitHub](#2-git--github)
- [Node.js](#3-nodejs)
- [Claude Code](#4-claude-code)
- [Cursor IDE](#5-cursor-ide)
- [First cloud accounts](#6-cloud-accounts-tier-1)

### Tier 2: Build real projects (next session)
- [Python](#7-python--virtual-environments)
- [PostgreSQL](#8-postgresql)
- [Codex CLI](#9-codex-cli)
- [Railway CLI](#10-railway-cli)
- [More cloud accounts](#11-cloud-accounts-tier-2)

### Tier 3: Full stack (when you need it)
- [Redis](#12-redis)
- [Docker](#13-docker)
- [Toolkit setup](#14-claude-toolkit)
- [Security tools](#15-security-tools)

### Tier 4: Skip for now
- Rust (only needed for Joyride backend)
- Ruby (only needed for iOS CocoaPods)
- Go (only needed for niche CLI tools)
- Mobile dev (Android Studio, Expo -- add later)
- Heroku, AWS (not used)

---

## Tier 1: Code TODAY

### 1. Windows Foundations + WSL2

WSL2 gives you a real Ubuntu Linux inside Windows. This is where you'll do 90% of your work.

```powershell
# Run these in PowerShell as Administrator

# 1. Install WSL2 with Ubuntu (default)
wsl --install

# 2. Restart your computer when prompted

# 3. After restart, Ubuntu will open and ask you to create a username/password
#    Use a simple username (lowercase, no spaces) -- this is your Linux user

# 4. Enable long file paths (prevents errors with node_modules)
reg add "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v "LongPathsEnabled" /t REG_DWORD /d 1 /f

# 5. Enable Developer Mode (for symlinks)
#    Settings > Privacy & Security > For Developers > Developer Mode: ON
```

After restart, open **Windows Terminal** (comes with Windows 11) and select the **Ubuntu** tab. This is your primary terminal from now on.

```bash
# Inside WSL2 Ubuntu -- update everything
sudo apt update && sudo apt upgrade -y

# Install essential build tools
sudo apt install -y build-essential curl wget jq unzip
```

**Important mental model:** Your Windows files live at `/mnt/c/Users/YourName/` inside WSL2. But for best performance, keep code projects inside WSL2's own filesystem at `~/projects/` (not on the Windows drive).

```bash
# Create your projects directory
mkdir -p ~/projects
```

### 2. Git + GitHub

```bash
# Inside WSL2

# Git is pre-installed in Ubuntu, but update it
sudo apt install -y git

# Configure (use your real name and GitHub email)
git config --global user.name "Don"
git config --global user.email "don@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.autocrlf input

# Install GitHub CLI
sudo apt install -y gh

# Or if the apt version is old:
# curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
# echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
# sudo apt update && sudo apt install gh

# Authenticate with GitHub (follow the prompts -- pick HTTPS and browser login)
gh auth login
```

### 3. Node.js

```bash
# Inside WSL2

# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# Reload your shell
source ~/.bashrc

# Install latest LTS
nvm install --lts

# Verify
node --version    # Should show v22.x or similar
npm --version     # Should show 10.x+

# Essential global packages
npm install -g pnpm          # Better package manager (used by some projects)
npm install -g tsx            # Run TypeScript files directly
```

### 4. Claude Code

```bash
# Inside WSL2
npm install -g @anthropic-ai/claude-code

# Verify
claude --version

# First run -- this will open a browser for Anthropic login
claude
```

Claude Code is your primary coding assistant. It runs in the terminal and can read/write files, run commands, and help you build anything.

**Quick orientation:**
- Type naturally to ask it to do things ("create a new Express app", "fix this error")
- It will ask permission before running commands or editing files
- `/help` shows available commands

### 5. Cursor IDE

Install on **Windows** (not inside WSL2 -- it's a GUI app):

1. Download from [cursor.com](https://cursor.com)
2. Run the installer
3. Open Cursor > open a **WSL2 folder**: `File > Open Folder` then type `\\wsl$\Ubuntu\home\yourname\projects`

Or install the **Remote - WSL** extension and open WSL folders directly.

**Recommended extensions** (install inside Cursor):
- ESLint
- Prettier
- Tailwind CSS IntelliSense
- GitLens
- Error Lens

### 6. Cloud Accounts (Tier 1)

Sign up for these now -- you'll need them immediately:

| Service | URL | Why | Free Tier? |
|---------|-----|-----|-----------|
| **GitHub** | github.com | Code hosting, everything | Yes |
| **Anthropic** | console.anthropic.com | Claude API key (for Claude Code pro features and apps) | Pay-as-you-go |
| **Vercel** | vercel.com | Easiest way to deploy web apps | Yes (hobby) |

After signing up, get your **Anthropic API key** and add it to your shell:

```bash
# Inside WSL2
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key-here"' >> ~/.bashrc
source ~/.bashrc
```

---

## Tier 2: Build Real Projects

### 7. Python + Virtual Environments

```bash
# Inside WSL2

# Python 3 is pre-installed on Ubuntu, but ensure it's up to date
sudo apt install -y python3 python3-pip python3-venv

# Verify
python3 --version    # Should show 3.12+

# Install pipx (for global CLI tools, isolated from projects)
sudo apt install -y pipx
pipx ensurepath
source ~/.bashrc
```

**Virtual Environments -- How They Work:**

Every Python project should have its own isolated environment. This prevents "dependency hell" where Project A needs library v1 and Project B needs library v2.

```bash
# The workflow for EVERY Python project:

# 1. Enter the project
cd ~/projects/my-app

# 2. Create a virtual environment (one-time per project)
python3 -m venv .venv

# 3. Activate it (do this every time you work on the project)
source .venv/bin/activate
# Your prompt changes to show (.venv)

# 4. Install the project's dependencies
pip install -r requirements.txt
# Or if the project uses pyproject.toml:
pip install -e ".[dev]"

# 5. Work on the project...

# 6. When done, deactivate
deactivate
```

**Key rules:**
- Never `pip install` without an active venv (you'll pollute the system Python)
- The `.venv/` folder is local to the project and gitignored
- If something breaks, just delete `.venv/` and recreate it

### 8. PostgreSQL

```bash
# Inside WSL2
sudo apt install -y postgresql postgresql-contrib

# Start the service
sudo service postgresql start

# Create your user (match your WSL username)
sudo -u postgres createuser --superuser $USER
createdb $USER

# Verify
psql -c "SELECT version();"

# SQLite is already included in Python and Ubuntu
sqlite3 --version
```

**Tip:** To auto-start PostgreSQL when WSL2 opens, add to your `~/.bashrc`:
```bash
# Auto-start PostgreSQL
if ! pg_isready -q 2>/dev/null; then
    sudo service postgresql start > /dev/null 2>&1
fi
```

To avoid the sudo password prompt each time:
```bash
# Run this once
echo "$USER ALL=(ALL) NOPASSWD: /usr/sbin/service postgresql start" | sudo tee /etc/sudoers.d/postgresql
```

### 9. Codex CLI

```bash
# Inside WSL2
npm install -g @openai/codex

# Add your OpenAI API key
echo 'export OPENAI_API_KEY="sk-your-key-here"' >> ~/.bashrc
source ~/.bashrc

# Verify
codex --version
```

### 10. Railway CLI

Railway is where Matt's apps are deployed. Simpler than AWS, more flexible than Vercel for backends.

```bash
# Inside WSL2
npm install -g @railway/cli

# Login
railway login

# Verify
railway --version
```

### 11. Cloud Accounts (Tier 2)

| Service | URL | Why | Free Tier? |
|---------|-----|-----|-----------|
| **OpenAI** | platform.openai.com | Codex CLI, GPT API | Pay-as-you-go |
| **Railway** | railway.app | Deploy backends + databases | $5/mo trial credit |
| **Docker Hub** | hub.docker.com | Container registry (for later) | Yes |

---

## Tier 3: Full Stack

### 12. Redis

```bash
# Inside WSL2
sudo apt install -y redis-server

# Start it
sudo service redis-server start

# Verify
redis-cli ping    # Should return PONG
```

Only needed for `x-business-intelligence` project (job queue).

### 13. Docker

Install **Docker Desktop** on Windows (not inside WSL2):

1. Download from [docker.com](https://docker.com/products/docker-desktop/)
2. Install -- it will enable WSL2 integration automatically
3. In Docker Desktop settings: `Resources > WSL Integration > Enable for Ubuntu`

```bash
# Verify from inside WSL2
docker --version
docker run hello-world
```

### 14. Claude Toolkit

Matt's toolkit adds commands like `/plan-feature`, `/implement-feature`, `/review-code` to Claude Code.

```bash
# Inside WSL2
cd ~/projects
git clone https://github.com/matthewliu/claude-toolkit.git
cd claude-toolkit

# Install MCP server deps
cd mcp-servers/review-server && npm install && cd ../..

# Deploy to your Claude Code config
./sync.sh push

# Enable the plugin -- edit ~/.claude/settings.json:
# Add "matt-toolkit": true to enabledPlugins
```

**Note:** `sync.sh` is a bash script and runs perfectly in WSL2. This is one of the reasons WSL2 is the right choice.

### 15. Security Tools

```bash
# Inside WSL2

# gitleaks (secret scanning)
# Download latest release binary:
GITLEAKS_VERSION=$(curl -s https://api.github.com/repos/gitleaks/gitleaks/releases/latest | jq -r .tag_name | tr -d 'v')
wget "https://github.com/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_amd64.tar.gz"
tar -xzf "gitleaks_${GITLEAKS_VERSION}_linux_amd64.tar.gz"
sudo mv gitleaks /usr/local/bin/
rm -f "gitleaks_${GITLEAKS_VERSION}_linux_amd64.tar.gz" LICENSE README.md

# semgrep (static analysis)
pipx install semgrep

# ruff (fast Python linter)
pipx install ruff
```

---

## Stack Recommendation for Beginners

**Start with JavaScript/Node.js for everything.** One language, frontend and backend. Here's the learning path:

```
Week 1:  Terminal basics, git, Claude Code
         Build a simple Express.js API
         Deploy to Vercel (or Railway for backend)

Week 2:  React frontend with Vite + Tailwind
         Connect frontend to your API
         PostgreSQL basics (or start with SQLite)

Week 3:  Python basics (for working on AgenticMiniApps)
         Virtual environments, pip, FastAPI
         Railway deploys

Week 4+: Pick a real project and build with Claude Code
```

**Vercel vs Railway:**
- **Vercel** = easiest deploys for frontend-only or Next.js apps. `vercel` and done.
- **Railway** = better for backends, databases, Python apps, anything with a Dockerfile. This is what Matt's projects use.
- **Start with Vercel** for your first deploys, switch to Railway when you need a database or Python backend.

---

## API Keys Summary

Add these to `~/.bashrc` as you sign up for services:

```bash
# Tier 1 (get immediately)
export ANTHROPIC_API_KEY="sk-ant-..."     # Claude Code + apps

# Tier 2 (get when needed)
export OPENAI_API_KEY="sk-..."            # Codex CLI

# Tier 3 (only for specific projects)
export GOOGLE_API_KEY="AI..."             # Gemini (toolkit reviews)
export XAI_API_KEY="xai-..."             # Grok (toolkit reviews)
```

Don't worry about Google, xAI, Groq, Perplexity, SendGrid, etc. until you're working on a specific project that needs them. Claude Code and Codex cover 95% of your needs.

---

## Verify Your Setup

After completing Tiers 1-2, run this checklist:

```bash
# Should all work from inside WSL2
echo "=== Core ==="
node --version          # v22+
npm --version           # 10+
python3 --version       # 3.12+
git --version           # 2.40+
gh --version            # 2.40+

echo "=== AI ==="
claude --version        # Claude Code
codex --version         # OpenAI Codex

echo "=== Databases ==="
psql --version          # PostgreSQL 16+
sqlite3 --version       # 3.x

echo "=== Package Managers ==="
pnpm --version

echo "=== Deployment ==="
railway version
```

---

## Troubleshooting

**WSL2 won't install:** Windows 11 Home supports WSL2 but you need to enable "Virtual Machine Platform" in Windows Features. Search "Turn Windows features on or off" in Start menu.

**`wsl` command not found:** Update Windows 11 to the latest version first.

**Cursor can't see WSL2 files:** Install the "Remote - WSL" extension in Cursor, then use `Ctrl+Shift+P > Remote-WSL: New Window`.

**PostgreSQL won't start:** `sudo service postgresql start`. If it fails, check logs: `sudo cat /var/log/postgresql/postgresql-*-main.log`.

**Node packages fail to install:** Usually a permissions issue. Never use `sudo npm install -g`. If npm's global directory has wrong permissions:
```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

**Python: "externally managed environment" error:** This means you need to use a venv. Don't use `--break-system-packages`. Always activate a venv first.
