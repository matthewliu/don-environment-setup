# Don's Windows 11 Dev Machine Setup

Setting up Windows 11 Home for coding with Claude Code, Cursor, and Matt's toolkit.

**Key decision: WSL2 is your primary dev environment.** Windows 11 Home supports WSL2 fully. You'll install GUI apps (Cursor, Docker Desktop) on Windows, but do all terminal work inside WSL2's Ubuntu. This gives you the same Linux environment Matt runs on Mac, so commands, scripts, and tutorials all work identically.

---

## The Strategy: Bootstrap, Then Let Claude Do the Rest

You need to manually set up 5 things before Claude Code can take over:
1. WSL2 (Linux inside Windows)
2. A GitHub account + git configured
3. Node.js
4. Claude Code
5. Clone this repo

Once Claude Code is running inside this repo, you tell it to read this file and finish the setup for you. It checks what's installed, installs what's missing, and explains everything along the way.

---

## Priority Order

### Tier 0: Bootstrap (30 minutes, manual -- follow every step)
- [WSL2 + GitHub + Node + Claude Code + clone this repo](#tier-0-bootstrap)

### Tier 1: Hand off to Claude
- [Let Claude finish the environment setup](#tier-1-hand-off-to-claude)

### Tier 2: Build real projects (tell Claude when ready)
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

Follow these steps in order. Copy-paste the commands exactly. Don't skip anything.

### Step 1: Install WSL2 + Ubuntu

WSL2 puts a real Linux (Ubuntu) inside Windows. This is where you'll write code.

1. Click the **Start menu**, type `PowerShell`, right-click **Windows PowerShell**, and choose **Run as administrator**.
2. Paste this command and press Enter:

```powershell
wsl --install
```

3. **Restart your computer** when it tells you to.
4. After restart, Ubuntu **might** open automatically. If it doesn't, open it yourself:
   - Click the **Start menu**, type `Ubuntu`, and click the **Ubuntu** app
   - If you don't see it, open **Microsoft Store**, search for "Ubuntu", install it, then launch it
   - **The Ubuntu download is large (~2-6 GB).** This is normal -- it's a full Linux operating system. While it downloads, jump to [Step 1B: While Ubuntu Downloads](#step-1b-while-ubuntu-downloads) below and come back here when it's done.
5. The first time Ubuntu opens, it asks you to create a username and password.
   - **Username:** pick something short and lowercase, like `don` (no spaces, no capitals)
   - **Password:** pick something you'll remember. When you type it, nothing shows on screen -- that's normal, just type and press Enter.
6. You're now at a Linux command prompt. It looks like `don@PCNAME:~$`. This is where you'll work.

**Tip:** For future sessions, open **Windows Terminal** (Start menu → type "Terminal"). It has a dropdown arrow next to the `+` tab button -- click it and select **Ubuntu** to get a Linux tab. You can set Ubuntu as the default profile in Terminal settings so it always opens Linux first.

Now skip ahead to [Step 2: Understand the WSL2 File System](#step-2-understand-the-wsl2-file-system).

### Step 1B: While Ubuntu Downloads

Do these while you wait. All of this is in your browser or Windows settings -- no Linux needed.

**1. Sign up for Anthropic (needed for Claude Code login):**
- Go to **console.anthropic.com** in your browser
- Click **Sign up** and create an account
- You'll need this to log in to Claude Code later
- Optionally, add a payment method and create an API key (go to **API Keys** in the left sidebar, click **Create Key**, and save it somewhere safe -- you'll need it later)

**2. Download Cursor (your code editor):**
- Go to **cursor.com** in your browser
- Click **Download** and run the installer
- Open Cursor after installing -- you can explore it, but we'll configure it for Linux later

**3. Apply Windows tweaks** (open PowerShell as Administrator for these):

Enable long file paths (prevents errors with deeply nested code folders):
```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v "LongPathsEnabled" /t REG_DWORD /d 1 /f
```

Enable Developer Mode (lets git create special file links called symlinks):
- Click the **Start menu** → click the **gear icon** (Settings)
- In the **search bar at the top of Settings**, type `Developer Mode`
- Click the result that says **Developer Mode** or **For developers**
- Toggle **Developer Mode** to **On**
- Click **Yes** on the confirmation popup
- (If you can't find it at all, don't worry -- skip this for now. It only matters for specific projects later, and Claude can help you find it.)

**4. Sign up for Vercel (free, for deploying web apps later):**
- Go to **vercel.com** in your browser
- Click **Sign Up**
- Choose **Continue with GitHub**
- GitHub will ask to authorize Vercel -- click **Authorize Vercel**
- Vercel will ask about a team/plan -- choose **Hobby** (free) and continue
- You're in. No credit card needed. When you deploy your first app later, Vercel auto-connects to your GitHub repos.

**Once Ubuntu finishes downloading, go back to Step 1, step 5** (creating your Linux username and password).

Run this to update everything and install basic tools:

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential curl wget unzip git
```

It will ask for your password (the one you just created). Type it and press Enter.

Create a folder for all your code projects:

```bash
mkdir -p ~/projects
```

**From now on, always use the Ubuntu tab in Windows Terminal for coding.** You can open Windows Terminal from the Start menu -- it has tabs for PowerShell and Ubuntu. Use the Ubuntu tab.

### Step 2: Understand the WSL2 File System

WSL2 creates a Linux filesystem alongside your Windows filesystem. Here's how they relate:

```
Windows sees:                         Linux (WSL2) sees:
C:\Users\Don\Documents\               /mnt/c/Users/Don/Documents/
C:\Users\Don\Desktop\                 /mnt/c/Users/Don/Desktop/

                                      /home/don/              ← your Linux home folder
                                      /home/don/projects/     ← where your code lives
```

**Two key things:**
- **From Linux**, you can access all your Windows files at `/mnt/c/`. So your Desktop is at `/mnt/c/Users/Don/Desktop/`.
- **From Windows**, you can browse your Linux files in File Explorer at `\\wsl$\Ubuntu\home\don\`. Just type that into the File Explorer address bar.

**Where should your code live?** In the Linux filesystem (`~/projects/`), not on the Windows drive. Why: the Linux filesystem is much faster for code operations (npm install, git, etc.). Files on `/mnt/c/` are slow because they cross between Windows and Linux on every read/write.

You can still open these files in Windows apps like Cursor -- you just point Cursor at `\\wsl$\Ubuntu\home\don\projects\` and it works.

### Step 3: Set Up Git and Connect to GitHub

You already have a GitHub account. Now you need to connect your terminal to it so you can download and upload code.

Back in the Ubuntu terminal, configure git with your name and the email you used for GitHub:

```bash
git config --global user.name "Don"
```

```bash
git config --global user.email "your-actual-github-email@example.com"
```

```bash
git config --global init.defaultBranch main
```

```bash
git config --global pull.rebase false
```

```bash
git config --global core.autocrlf input
```

Now install the GitHub CLI and log in:

```bash
sudo apt install -y gh
```

If that installs an old version (check with `gh --version` -- you want 2.50+), use this instead:

```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install gh
```

Now log in to GitHub from the terminal:

```bash
gh auth login
```

It will ask you questions. Choose these answers:
- **Where do you use GitHub?** → `GitHub.com`
- **What is your preferred protocol?** → `HTTPS`
- **Authenticate Git with your GitHub credentials?** → `Yes`
- **How would you like to authenticate?** → `Login with a web browser`

It will show you a one-time code (like `ABCD-1234`). Press Enter, and it opens your browser. Paste the code into the browser page, click **Authorize**, and you're connected.

Verify it worked:

```bash
gh auth status
```

You should see a green checkmark and your GitHub username.

### Step 4: Install Node.js

Node.js is the runtime for JavaScript. Claude Code and most web tools need it.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

```bash
source ~/.bashrc
```

```bash
nvm install --lts
```

Verify it worked:

```bash
node --version
```

You should see `v22.x.x` or similar. If you see `command not found`, close the terminal, reopen Ubuntu, and try `node --version` again.

### Step 5: Install Claude Code

Claude Code is an AI coding assistant that runs in your terminal. Think of it as a very smart pair programmer.

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:

```bash
claude --version
```

Now start it for the first time:

```bash
claude
```

It will open your browser to log in. You need an Anthropic account:
1. If you don't have one, go to **console.anthropic.com** and sign up
2. After logging in, the terminal will say you're authenticated
3. Type `/exit` to quit Claude for now -- we'll start it again in the right folder

### Step 6: Clone This Repo

"Cloning" means downloading a copy of a code project from GitHub to your computer. Run this exact command:

```bash
cd ~/projects && gh repo clone matthewliu/don-environment-setup
```

Verify it worked:

```bash
ls ~/projects/don-environment-setup
```

You should see `README.md`, `windows-setup.md`, and `mac-setup.md`.

**Tier 0 is complete.** You now have Linux, git, GitHub, Node.js, Claude Code, and this guide on your machine.

---

## Tier 1: Hand Off to Claude

Now start Claude Code inside this repo:

```bash
cd ~/projects/don-environment-setup && claude
```

Claude will start up. Paste this message to it:

> Read the file windows-setup.md. I just finished Tier 0 -- WSL2, git, GitHub, Node.js, and Claude Code are installed and working. I'm a beginner. Please do these things for me step by step:
>
> 1. Run the Windows tweaks listed in Tier 1 (tell me what to paste into PowerShell as admin)
> 2. Install the global npm packages listed (pnpm, tsx)
> 3. Help me pick and download an IDE (Cursor is recommended)
> 4. Check what's already installed and tell me if anything from Tier 0 needs fixing
>
> After that, explain what Tier 2 covers and ask me if I'm ready to continue or want to stop for today.

### What Claude will do:

**Windows tweaks** -- it will tell you to open PowerShell as Administrator and paste specific commands:
- Enable long file paths (prevents errors with deeply nested folders)
- Enable Developer Mode (lets git create symlinks)

**npm packages** -- it will run these for you:
```bash
npm install -g pnpm tsx
```

**Cursor IDE** -- it will tell you to:
1. Go to cursor.com in your browser and download the Windows installer
2. Run the installer
3. Open Cursor, install the "Remote - WSL" extension
4. Open your WSL2 projects folder: `File > Open Folder` → type `\\wsl$\Ubuntu\home\don\projects`

**Cloud accounts** -- it will walk you through what to sign up for:

| Service | URL | Why | Free? |
|---------|-----|-----|-------|
| **GitHub** | github.com | Already done in Tier 0 | Yes |
| **Anthropic** | console.anthropic.com | Already done (Claude Code login) | Pay-as-you-go |
| **Vercel** | vercel.com | Easiest way to deploy web apps to the internet | Yes (hobby tier) |

### How to set up Tier 2 later

When you're ready to install Python, PostgreSQL, and more, start Claude in this folder again and paste:

> Read windows-setup.md. I finished Tiers 0 and 1. Set up Tier 2 for me -- Python, PostgreSQL, Codex CLI, and Railway. Check what's already installed first and skip anything that's done. Explain what each thing is as you install it.

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
