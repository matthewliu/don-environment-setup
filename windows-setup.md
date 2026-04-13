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

### Tier 2: Install everything else (one big block)
- [Python, PostgreSQL, Codex, Railway, npm packages -- all at once](#tier-2-install-everything-else)

### Tier 3: Toolkit + security tools
- [Claude Toolkit](#tier-3-toolkit--security-tools)
- [Security tools (gitleaks, semgrep)](#security-tools)

### Tier 4: Skip for now (install when a project needs it)
- Docker
- Redis
- Rust (only needed for Joyride backend)
- Ruby (only needed for iOS CocoaPods)
- Go (only needed for niche CLI tools)
- Mobile dev (Android Studio, Expo -- add later)
- Heroku, AWS (not used)

---

## Tier 0: Bootstrap

Follow these steps in order. Copy-paste the commands exactly. Don't skip anything.

### Step 1: Install WSL2 + Ubuntu

WSL2 puts a real Linux (Ubuntu) inside Windows. This is where you'll write code. The install has two parts: first enable WSL2, then install Ubuntu.

#### Part A: Disable Smart App Control

This Windows 11 security feature blocks developer tools including Ubuntu, Node.js packages, and CLI tools. You need to turn it off first.

1. Open **Settings** (Start menu → gear icon)
2. Type `Smart App Control` in the search bar at the top
3. Click the result
4. Change it from **On** to **Off**
5. It will warn this can't be turned back on without resetting Windows -- that's fine for a dev machine, confirm it

#### Part B: Enable WSL2

1. Click the **Start menu**, type `PowerShell`
2. Right-click **Windows PowerShell** and choose **Run as administrator**
3. If a popup asks "Do you want to allow this app to make changes?" click **Yes**
4. Paste this command and press Enter:

```powershell
wsl --install --no-distribution
```

This enables the WSL2 feature without downloading Ubuntu yet (we'll do that separately so it's clearer).

5. Wait for it to finish. You'll see messages about enabling features.
6. **Restart your computer** when it tells you to.

#### Part C: Install Ubuntu

After your computer restarts:

1. Open **PowerShell as Administrator** again (Start menu → type `PowerShell` → right-click → **Run as administrator**)
2. Paste this command and press Enter:

```powershell
wsl --install -d Ubuntu
```

3. **The download is ~600 MB to 2 GB.** While it downloads, jump to [Step 1D: While Ubuntu Downloads](#step-1d-while-ubuntu-downloads) and come back here when it's done.
4. Once the download finishes and the install completes, it will either open Ubuntu automatically or tell you to launch it. If it doesn't open on its own, type this in PowerShell:

```powershell
ubuntu
```

**If the PowerShell install fails**, use the Microsoft Store as a fallback:
- Open **Microsoft Store** (Start menu → type `Microsoft Store`)
- Search for `Ubuntu`
- Click the one that just says **Ubuntu** (published by Canonical Group Limited)
- Click **Get** / **Install**, then **Open** when done

#### Part C (continued): First-time Ubuntu setup

When Ubuntu opens for the first time, it will spend a minute "Installing" (extracting files). Then it asks you to create a Linux user account:

```
Enter new UNIX username:
```

7. Type a short lowercase username like `don` (no spaces, no capitals) and press Enter
8. It asks for a password:

```
New password:
```

9. Type a password and press Enter. **Nothing shows on screen while you type -- no dots, no stars, nothing.** That's normal Linux behavior. Just type your password and press Enter.
10. It asks you to retype the password. Type the same password again and press Enter.
11. You'll see a message like `Installation successful!` and then a prompt that looks like:

```
don@YOURPC:~$
```

**You're in Linux.** This is your command line. You'll type commands here.

**Important -- copy/paste is different in the terminal:**

| Action | In Windows apps (normal) | In Ubuntu terminal |
|--------|------------------------|--------------------|
| Copy | `Ctrl+C` | `Ctrl+Shift+C` |
| Paste | `Ctrl+V` | `Ctrl+Shift+V` (or just **right-click**) |

`Ctrl+C` by itself means "cancel the running command" in a terminal, so you need to hold Shift too. You can also right-click anywhere in the terminal to paste from your clipboard. You'll use this constantly when copy-pasting commands from this guide.

**Linux gotchas for Windows users:**

| Gotcha | What it means |
|--------|--------------|
| **Case sensitive** | `cd Projects` and `cd projects` are DIFFERENT. File names, commands, everything cares about caps. |
| **Forward slashes** | Paths use `/` not `\`. It's `/home/don/projects/` not `\home\don\projects\`. |
| **Hidden files** | Files starting with `.` are invisible. `.bashrc`, `.env`, `.git` won't show with `ls`. Use `ls -a` to see them. |
| **`sudo`** | Means "run as admin." It asks for YOUR password (the one you just created). |
| **`~` (tilde)** | Shortcut for your home folder. `~/projects` = `/home/don/projects`. |
| **Tab to autocomplete** | Press Tab while typing a file/folder name to autocomplete it. Press Tab twice to see all matches. |
| **Up arrow** | Press up to scroll through your previous commands. Way faster than retyping. |
| **No undo for `rm`** | `rm` (delete) is permanent. There is no recycle bin. Be careful. |
| **"command not found"** | Means you need to install something, or restart your terminal. Claude Code can help. |

#### Part C (continued): Update Ubuntu and install build tools

Copy and paste this entire block into the Ubuntu terminal and press Enter:

```bash
sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential curl wget unzip git
```

It will ask for your password (the one you just created). Type it and press Enter. This downloads updates and essential tools -- takes a few minutes.

**A note about install commands:** Commands like `sudo apt install`, `npm install`, and `brew install` print a LOT of text as they work -- download progress, package names, sometimes yellow warnings. **This is all normal.** Just wait for it to finish (your `$` prompt comes back) before running the next command. Only worry if you see red error messages.

Then create a folder for your code:

```bash
mkdir -p ~/projects
```

#### How to open Ubuntu in the future

**Option 1 (easiest):** Click Start menu → type `Ubuntu` → click the app.

**Option 2 (better):** Use **Windows Terminal** which has tabs:
- Click Start menu → type `Terminal` → open **Terminal**
- Click the **small down arrow (v)** next to the **+** tab button at the top
- Click **Ubuntu** from the dropdown
- This gives you an Ubuntu tab alongside PowerShell tabs

**Tip:** To make Ubuntu the default tab, open Terminal → click the down arrow → **Settings** → under "Default profile" select **Ubuntu** → click **Save**.

Now continue to [Step 1D](#step-1d-while-ubuntu-downloads) if you haven't done it, or [Step 2](#step-2-understand-the-wsl2-file-system) if you have.

#### Step 1D: While Ubuntu Downloads

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

**What to expect:** npm will download a lot of files. You'll see progress bars, package names flying by, and possibly some yellow "WARN" messages. **Warnings are normal -- ignore them.** Only red "ERR!" messages are real problems. The install takes 1-2 minutes.

When it finishes, you'll see something like `added 123 packages` and your prompt comes back.

Verify it installed correctly:

```bash
claude --version
```

You should see a version number like `2.x.x`. If you see `command not found`, close the terminal, reopen Ubuntu, and try again.

Now start it for the first time:

```bash
claude
```

The first time you run Claude Code, it walks you through several setup screens:

1. **Terms of Service** -- it shows a wall of text. Press **Enter** to scroll through, then type `yes` and press Enter to accept.

2. **Login method** -- it asks how you want to log in. Choose one:
   - **Anthropic account (recommended)** -- if you signed up at console.anthropic.com during Step 1D
   - It will open your browser (it may print a URL -- if it doesn't open automatically, copy the URL and paste it into your browser). Log in, then authorize Claude Code.
   - Once authorized, the terminal will say you're authenticated.

3. **Choose your plan** -- it asks which plan to use. If you're on the Claude Pro/Max subscription, you can use that. Otherwise, use your API key.

4. **Permissions** -- it asks what Claude Code is allowed to do. For now, choose the defaults (you can change this later). It will ask things like:
   - Allow Claude to read files? → **Yes**
   - Allow Claude to run commands? → It will ask permission each time (safe default)

5. You should now see Claude Code's prompt -- a place where you can type messages to Claude.

6. Type `/exit` to quit Claude for now -- we'll start it again in the right folder.

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

**Cursor IDE** -- see the [Cursor 101](#cursor-101) section below for full install and setup.

**Cloud accounts** -- it will walk you through what to sign up for:

| Service | URL | Why | Free? |
|---------|-----|-----|-------|
| **GitHub** | github.com | Already done in Tier 0 | Yes |
| **Anthropic** | console.anthropic.com | Already done (Claude Code login) | Pay-as-you-go |
| **Vercel** | vercel.com | Easiest way to deploy web apps to the internet | Yes (hobby tier) |

### How to set up Tier 2 later

When you're ready, you can either follow the Tier 2 steps below yourself (they're copy-paste), or start Claude in this folder and paste:

> Read windows-setup.md. I finished Tiers 0 and 1. Walk me through Tier 2 -- install Python, PostgreSQL, Codex, Railway, and npm packages. Check what's already installed first and skip anything that's done. Explain what each thing is as you install it.

---

## Tier 2: Install Everything Else

This installs Python, PostgreSQL, Codex CLI, Railway CLI, and useful npm packages in one session. Run these in your Ubuntu terminal.

### Step 1: Install system packages (one command)

This installs Python, PostgreSQL, and developer tools all at once:

```bash
sudo apt install -y python3 python3-pip python3-venv pipx postgresql postgresql-contrib
```

This takes a few minutes. Wait for the `$` prompt to come back.

### Step 2: Configure Python and PostgreSQL

Set up pipx (for installing Python CLI tools):

```bash
pipx ensurepath
```

```bash
source ~/.bashrc
```

Start PostgreSQL and create your database:

```bash
sudo service postgresql start
```

```bash
sudo -u postgres createuser --superuser $USER
```

```bash
createdb $USER
```

Verify both work:

```bash
python3 --version
```

```bash
psql -c "SELECT version();"
```

If `psql` drops you into a prompt, press **Ctrl+D** to exit.

**Tip:** To auto-start PostgreSQL when WSL2 opens so you don't have to run `sudo service postgresql start` every time, run these two commands once:

```bash
echo "$USER ALL=(ALL) NOPASSWD: /usr/sbin/service postgresql start" | sudo tee /etc/sudoers.d/postgresql
```

```bash
echo -e '\n# Auto-start PostgreSQL\nif ! pg_isready -q 2>/dev/null; then\n    sudo service postgresql start > /dev/null 2>&1\nfi' >> ~/.bashrc
```

### Step 3: Install npm packages (one command)

This installs the Codex CLI, Railway CLI, and other useful tools:

```bash
npm install -g @openai/codex @railway/cli pnpm tsx
```

Wait for it to finish (yellow warnings = normal).

### Step 4: Log in to Railway

```bash
railway login
```

This opens your browser. Sign up at **railway.app** if you don't have an account (use your GitHub account -- easiest), then authorize.

### Step 5: Add API keys

```bash
echo 'export OPENAI_API_KEY="sk-your-key-here"' >> ~/.bashrc
```

```bash
source ~/.bashrc
```

You need an OpenAI API key for Codex. Sign up at **platform.openai.com**, go to **API Keys**, create one, and paste it in the command above (replacing `sk-your-key-here`).

### Step 6: Verify everything

```bash
python3 --version
```

```bash
psql --version
```

```bash
codex --version
```

```bash
railway version
```

```bash
pnpm --version
```

```bash
sqlite3 --version
```

SQLite is already included with Ubuntu -- no install needed.

### Python Virtual Environments (Important Concept)

Every Python project should have its own isolated environment. This prevents conflicts where Project A needs library v1 and Project B needs library v2.

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

### Cloud Accounts (Tier 2)

| Service | URL | Why | Free? |
|---------|-----|-----|-------|
| **OpenAI** | platform.openai.com | Codex CLI, GPT API | Pay-as-you-go |
| **Railway** | railway.app | Deploy backends + databases | $5/mo trial credit |

---

## Tier 3: Toolkit + Security Tools

### Claude Toolkit

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

Now enable the plugin. The easiest way is to ask Claude Code to do it:

```bash
cd ~/projects/claude-toolkit && claude
```

Then tell Claude:

> Add "matt-toolkit": true to the enabledPlugins object in ~/.claude/settings.json

**Note:** `sync.sh` is a bash script and runs perfectly in WSL2. This is one of the reasons WSL2 is the right choice.

### Security Tools

```bash
pipx install semgrep
```

```bash
pipx install ruff
```

Install gitleaks (secret scanning):

```bash
GITLEAKS_VERSION=$(curl -s https://api.github.com/repos/gitleaks/gitleaks/releases/latest | jq -r .tag_name | tr -d 'v')
```

```bash
wget "https://github.com/gitleaks/gitleaks/releases/download/v${GITLEAKS_VERSION}/gitleaks_${GITLEAKS_VERSION}_linux_amd64.tar.gz"
```

```bash
tar -xzf "gitleaks_${GITLEAKS_VERSION}_linux_amd64.tar.gz" && sudo mv gitleaks /usr/local/bin/ && rm -f "gitleaks_${GITLEAKS_VERSION}_linux_amd64.tar.gz" LICENSE README.md
```

---

## Tier 4: Skip for Now

Install these only when a specific project needs them. Ask Claude Code if you're unsure.

- **Docker** -- for containerized deploys. Install Docker Desktop on Windows, enable WSL2 integration.
- **Redis** -- in-memory database. Only needed for x-business-intelligence. `sudo apt install -y redis-server`
- **Rust** -- only for Joyride backend. `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
- **Ruby** -- only for iOS CocoaPods.
- **Mobile dev** -- Android Studio, Expo, Xcode.
- **Heroku, AWS** -- not used.

---

## Cursor 101

Cursor is your code editor -- it's where you'll browse, read, and edit code files. It's based on VS Code but has built-in AI features.

### Install Cursor (Windows)

1. Go to **cursor.com** in your browser
2. Click **Download for Windows**
3. Run the installer
4. Open Cursor when it's done

### Connect Cursor to WSL2

Your code lives in WSL2's Linux filesystem, so Cursor needs to connect to it:

1. Open Cursor
2. Press **Ctrl+Shift+X** to open the Extensions panel (left sidebar)
3. Search for `WSL` and install **WSL** (by Microsoft)
4. Press **Ctrl+Shift+P** to open the Command Palette
5. Type `WSL: Connect to WSL` and click it
6. Cursor will reload and connect to your Ubuntu. You'll see **WSL: Ubuntu** in the bottom-left corner.
7. Now open your projects: **File > Open Folder** → navigate to `/home/don/projects` → click **OK**

**From now on, always connect to WSL first** before opening folders. If you see `WSL: Ubuntu` in the bottom-left, you're connected.

### Install Extensions

Press **Ctrl+Shift+X** to open Extensions, then search for and install each of these:

| Extension | What it does |
|-----------|-------------|
| **Python** (Microsoft) | Python syntax, linting, debugging, IntelliSense |
| **Pylance** (Microsoft) | Fast Python type checking (installs with Python extension) |
| **ESLint** | JavaScript/TypeScript error checking |
| **Prettier** | Auto-formats your code to look clean |
| **Tailwind CSS IntelliSense** | Autocomplete for Tailwind CSS classes |
| **GitLens** | See who changed each line, browse git history |
| **Error Lens** | Shows errors inline (next to the code, not just in a panel) |
| **GitHub Pull Requests** | Create and review PRs from inside Cursor |
| **Path Intellisense** | Autocompletes file paths when you type |

### Key Cursor Shortcuts

| What | Shortcut |
|------|----------|
| Open a file by name | `Ctrl+P` then type the filename |
| Search across all files | `Ctrl+Shift+F` |
| Open the terminal inside Cursor | `` Ctrl+` `` (backtick, the key above Tab) |
| Open Command Palette | `Ctrl+Shift+P` |
| Go to a line number | `Ctrl+G` |
| Toggle sidebar | `Ctrl+B` |
| Split editor (side by side) | `Ctrl+\` |
| AI chat (Cursor feature) | `Ctrl+L` |

### Configure Prettier to Format on Save

1. Press **Ctrl+Shift+P**
2. Type `Preferences: Open Settings (JSON)` and click it
3. Add these lines inside the `{}` braces:

```json
"editor.formatOnSave": true,
"editor.defaultFormatter": "esbenp.prettier-vscode"
```

4. Save the file (**Ctrl+S**)

Now every time you save a file, Prettier will auto-format it.

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

After completing Tiers 0-2, run each of these in your Ubuntu terminal:

```bash
node --version
```

```bash
python3 --version
```

```bash
git --version
```

```bash
gh auth status
```

```bash
claude --version
```

```bash
codex --version
```

```bash
psql --version
```

```bash
pnpm --version
```

```bash
railway version
```

If any say `command not found`, tell Claude Code and it will help you fix it.

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
