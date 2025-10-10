# Pyenv Crash Course: Manage Multiple Python Versions Like a Pro (macOS, Linux, Windows)

If you’re a developer juggling multiple Python projects, chances are you’ve hit the “wrong Python version” wall. One project needs Python 3.7, another needs 3.11 — and your system Python just isn’t cutting it.

Enter Pyenv — the ultimate tool to install, manage, and switch between multiple Python versions seamlessly.

## In this crash course, we’ll cover

- ✅ What Pyenv is and why you need it
- ⚙️ How to install and set it up on macOS, Linux, and Windows
- 🧠 Common commands and workflows
- 🧩 Integrations with virtual environments
- 💡 Troubleshooting tips

Let’s dive in.

---

## What is Pyenv?

Pyenv is a lightweight Python version management tool that lets you:

- Install multiple Python versions side-by-side
- Switch versions globally, per-shell, or per-project
- Avoid conflicts with your system Python

Think of it as Node Version Manager (nvm) — but for Python.

💡 Pyenv doesn’t replace virtual environments — it works with them. You use Pyenv to manage Python installations, and venv or virtualenv to manage dependencies.

---

## How Pyenv Works (Under the Hood)

When you install Pyenv, it:

1. Inserts a shim layer in your $PATH before the system Python.
2. Redirects any call to python, pip, etc., to the correct Python version you’ve set.

Example:

```bash
$ pyenv global 3.11.6
$ python --version
Python 3.11.6
```

Magically, all Python commands now use your specified version — no system hacks needed.

---

## Installation Guide

Let’s go OS by OS.

---

## macOS Setup

### Step 1: Install dependencies

Use Homebrew:

```bash
brew update
brew install pyenv
```

### Step 2: Add Pyenv to your shell

For zsh (default on macOS):

```bash
echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

Then restart your terminal.

### Step 3: Install Python versions

```bash
pyenv install 3.11.6
pyenv install 3.9.18
```

### Step 4: Set your global (default) version

```bash
pyenv global 3.11.6
```

---

## Linux Setup

### Step 1: Install dependencies using Package Manager

For Ubuntu/Debian:

```bash
sudo apt update
sudo apt install -y build-essential curl git \
    libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
    libsqlite3-dev wget llvm libncurses5-dev libffi-dev \
    liblzma-dev python3-openssl tk-dev
```

### Step 2: Install Pyenv

```bash
curl https://pyenv.run | bash
```

### Step 3: Add Pyenv to your shell

```bash
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init --path)"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
```

### Step 4: Install Python

```bash
pyenv install 3.10.14
pyenv global 3.10.14
```

---

## Windows Setup (via pyenv-win)

For Windows, Pyenv is available as pyenv-win.

### Step 1: Install via PowerShell

Run PowerShell as Administrator:

```bash
Invoke-WebRequest -UseBasicParsing -Uri "https://pyenv.run" -OutFile "./pyenv-win/install.ps1"
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
./pyenv-win/install.ps1
```

### Step 2: Add Pyenv to PATH

Make sure these are in your environment variables:

```bash
%USERPROFILE%\.pyenv\pyenv-win\bin
%USERPROFILE%\.pyenv\pyenv-win\shims
```

### Step 3: Install a Python version

```bash
pyenv install 3.11.5
pyenv global 3.11.5
```

Then verify:

```bash
python --version
```

---

## Common Pyenv Commands

| Command                 | Description                                    |
| ----------------------- | ---------------------------------------------- |
| pyenv install --list    | Show all available Python versions             |
| pyenv install 3.11.6    | Install a specific version                     |
| pyenv uninstall 3.10.14 | Remove a version                               |
| pyenv versions          | List installed versions                        |
| pyenv global 3.11.6     | Set global (system-wide) version               |
| pyenv local 3.9.18      | Set version for the current project            |
| pyenv shell 3.8.10      | Temporarily set version for this shell session |

---

## Bonus: Pyenv + Virtualenv

You can combine Pyenv with pyenv-virtualenv for per-project environments:

```bash
brew install pyenv-virtualenv
pyenv virtualenv 3.11.6 myenv
pyenv activate myenv
```

Deactivate with:

```bash
pyenv deactivate
```

This is great for isolating project dependencies while keeping version control simple.

---

## Troubleshooting

```bash
“Command not found: pyenv”
```

Make sure you’ve added Pyenv to your shell profile (.zshrc, .bashrc, etc.) and restarted the terminal.

```bash
“Python build failed”
```

Check dependencies. On Ubuntu:

```bash
sudo apt install -y build-essential libssl-dev zlib1g-dev
```

Can’t switch versions?

Run:

```bash
pyenv rehash
```

This rebuilds shims and fixes PATH conflicts.

---

## Integrate with IDEs

In VS Code, set your interpreter:

Command Palette → Python: Select Interpreter → Choose from pyenv versions

PyCharm, Sublime Text, and others detect Pyenv-managed interpreters automatically.

---

## Conclusion

Managing Python versions manually is a recipe for chaos. With Pyenv, you get:

- Clean version isolation
- Seamless switching
- A simple, developer-friendly workflow

Whether you’re on macOS, Linux, or Windows — Pyenv is the easiest way to stay sane across projects.

🚀 Start managing Python like a pro — your future self (and your CI/CD pipeline) will thank you.

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀