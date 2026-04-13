# Don's Windows 11 Dev Machine Setup

Setting up Windows 11 Home for coding with Claude Code, Cursor, and Matt's toolkit.

**Key decision: WSL2 is your primary dev environment.** Windows 11 Home supports WSL2 fully. You'll install GUI apps (Cursor, Docker Desktop) on Windows, but do all terminal work inside WSL2's Ubuntu. This gives you the same Linux environment Matt runs on Mac, so commands, scripts, and tutorials all work identically.

---

## The Strategy: Bootstrap, Then Let Claude Do the Rest

You only need to manually install **3 things**:
1. WSL2 (Linux inside Windows)
2. Node.js
3. Claude Code

That's it. Once Claude Code is running, you clone this repo and tell Claude: "set me up." It reads this guide, checks what's installed, and handles everything else -- git config, GitHub auth, Cursor, Python, databases, API keys, all of it.

---

## Priority Order

### Tier 0: Bootstrap (15 minutes, manual)
- [WSL2 + Node + Claude Code](#tier-0-bootstrap)

### Tier 1: Hand off to Claude
- [Clone this repo, Claude does the rest](#tier-1-hand-off-to-claude)

### Tier 2: Build real projects (Claude helps when needed)
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

## Tier 0: Bootstrap

Do these 3 steps manually. Copy-paste each block into the terminal. After this, Claude takes over.

### Step 1: WSL2

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

**Restart your computer.** After restart, Ubuntu opens automatically. It asks for a username and password -- pick a simple lowercase username (no spaces).

Once you're at the Ubuntu prompt, run this one block:

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential curl wget unzip && mkdir -p ~/projects
```

### Step 2: Node.js

Still in the Ubuntu terminal:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash && source ~/.bashrc && nvm install --lts
```

Verify: `node --version` should print `v22.x` or similar.

### Step 3: Claude Code

```bash
npm install -g @anthropic-ai/claude-code && claude --version
```

Now run `claude` -- it will open a browser to log in to your Anthropic account. Once authenticated, you have a coding AI in your terminal.

**You're done with manual setup.**

---

## Tier 1: Hand Off to Claude

Still in WSL2 Ubuntu. Tell Claude to clone this repo and set you up:

```bash
cd ~/projects
claude
```

Then paste this into Claude:

> Run `sudo apt install -y git gh` then configure git with my name "Don" and email "don@example.com". Then run `gh auth login` so I can authenticate with GitHub. After that, clone matthewliu/don-environment-setup into ~/projects. Then read windows-setup.md and walk me through the rest of the setup -- Cursor IDE, cloud accounts, Windows tweaks, and global npm packages. Check what's already installed and skip anything that's done.

Claude will handle:

**Git + GitHub** -- configures git, authenticates `gh`, clones repos.

**Windows tweaks** -- tells you what to run in PowerShell as Administrator:
- Enable long file paths (prevents node_modules errors)
- Enable Developer Mode (for symlinks)

**Cursor IDE** -- tells you to download from cursor.com (Windows app, not WSL2) and install the Remote-WSL extension for opening WSL2 folders.

**Cloud accounts** -- walks you through signing up for what you need:

| Service | URL | Why | Free Tier? |
|---------|-----|-----|-----------|
| **GitHub** | github.com | Code hosting, everything | Yes |
| **Anthropic** | console.anthropic.com | Claude API key (for apps you build) | Pay-as-you-go |
| **Vercel** | vercel.com | Easiest way to deploy web apps | Yes (hobby) |

**API keys + shell config:**
```bash
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key-here"' >> ~/.bashrc
source ~/.bashrc
```

**Global npm packages:**
```bash
npm install -g pnpm tsx
```

### Asking Claude for more

Once Tier 1 is done, you can ask Claude to set up Tier 2 whenever you're ready:

> Read windows-setup.md. Set up Tier 2 for me -- Python, PostgreSQL, Codex CLI, and Railway. Check what's already installed first.

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
