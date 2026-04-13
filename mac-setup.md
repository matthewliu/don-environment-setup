# Mac Setup Guide

Setting up a fresh Mac (Apple Silicon) for coding with Claude Code, Cursor, and Matt's toolkit.
Written for someone who has never coded before.

---

## The Strategy: Bootstrap, Then Let Claude Do the Rest

You need to manually set up 5 things:
1. Xcode Command Line Tools + Homebrew (Mac's package manager)
2. A GitHub account + git configured
3. Node.js
4. Claude Code
5. Clone the setup repo

Once Claude Code is running inside this repo, you tell it to read this file and finish the setup. It installs everything else for you.

---

## Priority Order

### Tier 0: Bootstrap (30 minutes, manual -- follow every step)
- [Xcode + Homebrew + GitHub + Node + Claude Code + clone this repo](#tier-0-bootstrap)

### Tier 1: Hand off to Claude
- [Let Claude finish the environment setup](#tier-1-hand-off-to-claude)

### Tier 2: Build real projects (tell Claude when ready)
- [Python](#7-python)
- [PostgreSQL](#8-postgresql)
- [Codex CLI](#9-codex-cli)
- [Railway CLI](#10-railway-cli)
- [More cloud accounts](#11-cloud-accounts-tier-2)

### Tier 3: Full stack (when you need it)
- [Redis](#12-redis)
- [Docker](#13-docker)
- [Toolkit setup](#14-claude-toolkit)
- [Security tools](#15-security-tools)
- [Utility CLIs](#16-utility-clis)

### Tier 4: Skip for now
- Rust (only needed for Joyride backend)
- Ruby (only needed for iOS CocoaPods)
- Mobile dev (Android Studio, Expo, Xcode)
- Heroku, AWS (not used)

---

## Tier 0: Bootstrap

Follow these steps in order. Don't skip anything.

### Step 1: Open Terminal

Terminal is the app where you type commands. It comes with every Mac.

1. Press **Cmd + Space** to open Spotlight search
2. Type `Terminal` and press Enter
3. A white (or black) window appears with a blinking cursor. This is your command line.

**Tip:** Right-click the Terminal icon in your Dock and choose **Options > Keep in Dock** so you can find it easily later.

**Important -- how to run commands from this guide:**
- Copy a command from this guide (highlight it, then **Cmd+C**)
- Click inside the Terminal window
- Paste it (**Cmd+V**)
- Press **Enter** to run it
- Wait for it to finish before running the next one. You'll know it's done when you see your prompt again (the `$` or `%` sign with a blinking cursor).

### Step 2: Install Xcode Command Line Tools

These are basic developer tools that almost everything else needs. Paste this into Terminal and press Enter:

```bash
xcode-select --install
```

A popup will appear asking if you want to install the tools. Click **Install**, then **Agree** to the license.

**This takes 5-15 minutes.** A progress bar will show. While it installs, you can read ahead but don't run any more commands until it finishes.

When it's done, you'll see `Install complete` or the Terminal prompt comes back.

### Step 3: Install Homebrew

Homebrew is a package manager for Mac -- it lets you install developer tools with simple commands instead of hunting for downloads. Paste this into Terminal:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

It will ask for your Mac password (the one you use to log in). **When you type it, nothing shows on screen -- no dots, no stars.** That's normal. Just type your password and press Enter.

It may also ask you to press Enter to confirm. Do it.

**This takes 5-10 minutes.** When it finishes, it will print some instructions. The important one is adding Homebrew to your PATH. Run these two commands:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
```

```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Verify it worked:

```bash
brew --version
```

You should see something like `Homebrew 4.x.x`. If you see `command not found`, close Terminal, reopen it, and try again.

### Step 4: Create a GitHub Account

GitHub is where all code is stored and shared. You need an account.

1. Open Safari (or your browser) and go to **github.com**
2. Click **Sign up**
3. Follow the steps -- use your real email, pick a username you like
4. Verify your email when GitHub sends you a confirmation

**Keep the browser open -- you'll need it in the next step.**

If you already have a GitHub account, skip this step.

### Step 5: Install Git and Connect to GitHub

Back in Terminal, install Git and the GitHub CLI:

```bash
brew install git gh
```

**A note about install commands:** Both `brew install` and `npm install` print a lot of text as they work -- download progress, package names, sometimes yellow warnings. **This is all normal.** Just wait for it to finish (your `$` or `%` prompt comes back) before running the next command. Only worry if you see red error messages.

Now configure git with your name and the email you used for GitHub:

```bash
git config --global user.name "Your Name"
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

### Step 6: Install Node.js

Node.js is the runtime for JavaScript. Claude Code and most web tools need it.

```bash
brew install node
```

This takes a minute or two. Verify it worked:

```bash
node --version
```

You should see `v22.x.x` or `v25.x.x`. Any version 20+ is fine.

```bash
npm --version
```

You should see `10.x` or `11.x`. npm is the package manager that comes with Node.

### Step 7: Install Claude Code

Claude Code is an AI coding assistant that runs in your terminal.

```bash
npm install -g @anthropic-ai/claude-code
```

**What to expect:** npm will download a lot of files. You'll see progress bars, package names flying by, and possibly some yellow "WARN" messages. **Warnings are normal -- ignore them.** Only red "ERR!" messages are real problems. The install takes 1-2 minutes.

When it finishes, you'll see something like `added 123 packages` and your prompt comes back.

Verify it installed correctly:

```bash
claude --version
```

You should see a version number like `2.x.x`. If you see `command not found`, close Terminal, reopen it, and try again.

Now start it for the first time:

```bash
claude
```

The first time you run Claude Code, it walks you through several setup screens:

1. **Terms of Service** -- it shows a wall of text. Press **Enter** to scroll through, then type `yes` and press Enter to accept.

2. **Login method** -- it asks how you want to log in. Choose one:
   - **Anthropic account (recommended)** -- if you signed up at console.anthropic.com
   - It will open your browser. Log in (or sign up if you haven't), then authorize Claude Code.
   - Once authorized, the terminal will say you're authenticated.

3. **Choose your plan** -- it asks which plan to use. If you're on the Claude Pro/Max subscription, you can use that. Otherwise, use your API key.

4. **Permissions** -- it asks what Claude Code is allowed to do. For now, choose the defaults (you can change this later). It will ask things like:
   - Allow Claude to read files? → **Yes**
   - Allow Claude to run commands? → It will ask permission each time (safe default)

5. You should now see Claude Code's prompt -- a place where you can type messages to Claude.

6. Type `/exit` to quit Claude for now -- we'll start it again in the right folder.

### Step 8: Create a Projects Folder and Clone This Repo

"Cloning" means downloading a copy of a code project from GitHub to your computer.

First, create a folder for all your code projects:

```bash
mkdir -p ~/projects
```

Now clone this setup repo:

```bash
cd ~/projects && gh repo clone matthewliu/don-environment-setup
```

Verify it worked:

```bash
ls ~/projects/don-environment-setup
```

You should see `README.md`, `windows-setup.md`, and `mac-setup.md`.

**Tier 0 is complete.** You now have a terminal, git, GitHub, Node.js, Claude Code, and this guide on your machine.

---

## Tier 1: Hand Off to Claude

Now start Claude Code inside this repo:

```bash
cd ~/projects/don-environment-setup && claude
```

Claude will start up. Paste this message to it:

> Read the file mac-setup.md. I just finished Tier 0 -- Terminal, Homebrew, git, GitHub, Node.js, and Claude Code are installed and working. I'm a beginner who has never coded before. Please do these things for me step by step:
>
> 1. Install the global npm packages listed in Tier 1 (pnpm, tsx)
> 2. Help me download and set up Cursor (my code editor)
> 3. Help me sign up for Vercel (for deploying web apps later)
> 4. Check what's already installed and tell me if anything from Tier 0 needs fixing
>
> After that, explain what Tier 2 covers and ask me if I'm ready to continue or want to stop for today.

### What Claude will do:

**npm packages** -- it will run these for you:
```bash
npm install -g pnpm tsx
```

**Cursor IDE:**
1. Go to **cursor.com** in your browser
2. Click **Download for Mac**
3. Open the downloaded `.dmg` file
4. Drag **Cursor** into your **Applications** folder
5. Open Cursor from Applications (or Spotlight: Cmd+Space → type `Cursor`)
6. If Mac says "Cursor is from an unidentified developer" → go to **System Settings > Privacy & Security** → scroll down → click **Open Anyway**
7. Inside Cursor, install these extensions (click the Extensions icon in the left sidebar, search for each):
   - ESLint
   - Prettier
   - Tailwind CSS IntelliSense
   - GitLens
   - Error Lens

**Cloud accounts:**

| Service | URL | Why | Free? |
|---------|-----|-----|-------|
| **GitHub** | github.com | Already done in Tier 0 | Yes |
| **Anthropic** | console.anthropic.com | Already done (Claude Code login) | Pay-as-you-go |
| **Vercel** | vercel.com | Easiest way to deploy web apps | Yes (hobby tier) |

**Vercel signup:**
- Go to **vercel.com** in your browser
- Click **Sign Up**
- Choose **Continue with GitHub**
- GitHub will ask to authorize Vercel -- click **Authorize Vercel**
- Choose **Hobby** (free) plan
- You're in. No credit card needed.

### How to set up Tier 2 later

When you're ready, start Claude in this folder again and paste:

> Read mac-setup.md. I finished Tiers 0 and 1. Set up Tier 2 for me -- Python, PostgreSQL, Codex CLI, and Railway. Check what's already installed first and skip anything that's done. Explain what each thing is as you install it.

---

## Tier 2: Build Real Projects

### 7. Python

Python is a programming language used for backends, AI, data science. Many of Matt's projects use it.

```bash
brew install python@3.12 pipx virtualenv
```

```bash
pipx ensurepath
```

Close and reopen Terminal (or run `source ~/.zshrc`) for the PATH change to take effect.

Verify:

```bash
python3 --version
```

You should see `Python 3.12.x`.

**Virtual Environments -- Why They Matter:**

Every Python project should have its own isolated environment. This prevents conflicts where Project A needs library v1 and Project B needs library v2.

```bash
# The workflow for EVERY Python project:

# 1. Enter the project folder
cd ~/projects/some-python-app

# 2. Create a virtual environment (one-time per project)
python3 -m venv .venv

# 3. Activate it (do this every time you work on the project)
source .venv/bin/activate
# Your prompt changes to show (.venv)

# 4. Install the project's dependencies
pip install -r requirements.txt

# 5. Work on the project...

# 6. When done, deactivate
deactivate
```

### 8. PostgreSQL

PostgreSQL is a database -- where apps store their data (users, posts, settings, etc.).

```bash
brew install postgresql@16
```

```bash
brew services start postgresql@16
```

```bash
echo 'export PATH="/opt/homebrew/opt/postgresql@16/bin:$PATH"' >> ~/.zshrc
```

```bash
source ~/.zshrc
```

```bash
createdb $(whoami)
```

Verify:

```bash
psql -c "SELECT version();"
```

You should see a PostgreSQL version line. Type `\q` or press **Ctrl+D** to exit if it drops you into a `psql` prompt.

SQLite (a simpler database for development) is already built into macOS:

```bash
sqlite3 --version
```

### 9. Codex CLI

Codex is OpenAI's coding assistant (similar to Claude Code but uses GPT models).

```bash
npm install -g @openai/codex
```

You'll need an OpenAI API key to use it. Sign up at **platform.openai.com**, then:

```bash
echo 'export OPENAI_API_KEY="sk-your-key-here"' >> ~/.zshrc
```

```bash
source ~/.zshrc
```

### 10. Railway CLI

Railway is a platform for deploying apps to the internet. Matt's projects use it.

```bash
brew install railway
```

```bash
railway login
```

This opens your browser. Sign up at **railway.app** if you don't have an account (sign up with GitHub for easiest setup), then authorize.

### 11. Cloud Accounts (Tier 2)

| Service | URL | Why | Free? |
|---------|-----|-----|-------|
| **OpenAI** | platform.openai.com | Codex CLI, GPT API | Pay-as-you-go |
| **Railway** | railway.app | Deploy backends + databases | $5/mo trial credit |

---

## Tier 3: Full Stack

### 12. Redis

Redis is an in-memory database used for caching and job queues. Only one project (x-business-intelligence) needs it.

```bash
brew install redis
```

```bash
brew services start redis
```

Verify:

```bash
redis-cli ping
```

Should print `PONG`.

### 13. Docker

Docker runs apps in containers (like lightweight virtual machines). Needed for deploying some projects.

1. Go to **docker.com/products/docker-desktop** in your browser
2. Click **Download for Mac (Apple Silicon)**
3. Open the downloaded `.dmg` file
4. Drag **Docker** into your **Applications** folder
5. Open Docker Desktop from Applications
6. It will ask for your Mac password -- enter it
7. Wait for Docker to start (you'll see a whale icon in the menu bar at the top of your screen)

Verify in Terminal:

```bash
docker --version
```

### 14. Claude Toolkit

Matt's toolkit adds commands like `/plan-feature`, `/implement-feature`, `/review-code` to Claude Code.

```bash
cd ~/projects
```

```bash
gh repo clone matthewliu/claude-toolkit
```

```bash
cd claude-toolkit
```

```bash
cd mcp-servers/review-server && npm install && cd ../..
```

```bash
./sync.sh push
```

Now you need to enable the plugin. Open the settings file:

```bash
open ~/.claude/settings.json
```

This opens the file in TextEdit. Find the `"enabledPlugins"` section and add `"matt-toolkit": true` inside the curly braces. Save and close.

(If you're not sure how to edit JSON, ask Claude Code to do it for you.)

### 15. Security Tools

```bash
brew install gitleaks
```

```bash
pipx install semgrep
```

### 16. Utility CLIs

```bash
brew install ripgrep jq tmux watch overmind coreutils openssl
```

---

## Tier 4: When Needed

Don't install these unless you're working on a project that specifically needs them. Ask Claude if you're unsure.

### Rust (Joyride backend only)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Press Enter when it asks about default installation. Then:

```bash
source $HOME/.cargo/env
```

### Ruby + iOS (building iOS apps locally)

```bash
brew install rbenv ruby-build
```

```bash
rbenv install 3.2.0
```

```bash
rbenv global 3.2.0
```

```bash
gem install cocoapods
```

### Mobile Dev

```bash
brew install watchman
```

```bash
brew install --cask android-studio
```

For iOS, install **Xcode** from the Mac App Store (it's free but ~12 GB), then:

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

```bash
sudo xcodebuild -runFirstLaunch
```

---

## Mac Terminal Cheat Sheet

| What you want to do | Command | Example |
|---------------------|---------|---------|
| See where you are | `pwd` | Shows `/Users/yourname/projects` |
| List files here | `ls` | Shows files in current folder |
| List ALL files (including hidden) | `ls -a` | Shows `.env`, `.git`, etc. |
| Go into a folder | `cd foldername` | `cd projects` |
| Go up one folder | `cd ..` | Goes to parent folder |
| Go to home folder | `cd ~` | Goes to `/Users/yourname` |
| Create a folder | `mkdir foldername` | `mkdir my-app` |
| Delete a file | `rm filename` | `rm old-file.txt` (permanent!) |
| Clear the screen | `clear` | Or press **Cmd+K** |
| Cancel a running command | **Ctrl+C** | Stops whatever is running |
| Search previous commands | **Up arrow** | Press up to scroll through history |
| Autocomplete file names | **Tab** | Press Tab while typing a name |

---

## API Keys Summary

Add these to `~/.zshrc` as you sign up for services. Run each line in Terminal:

```bash
# Tier 1 (get immediately)
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key"' >> ~/.zshrc
```

```bash
# Tier 2 (get when needed)
echo 'export OPENAI_API_KEY="sk-your-key"' >> ~/.zshrc
```

After adding keys, always run:

```bash
source ~/.zshrc
```

Don't worry about Google, xAI, Groq, Perplexity, SendGrid, etc. until you're working on a specific project that needs them.

---

## Verify Your Setup

After completing Tiers 0-2, run these in Terminal:

```bash
node --version
```

```bash
npm --version
```

```bash
python3 --version
```

```bash
git --version
```

```bash
gh --version
```

```bash
claude --version
```

```bash
psql --version
```

```bash
sqlite3 --version
```

If any of these say `command not found`, tell Claude Code and it will help you fix it.
