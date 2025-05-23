---
title: Getting fnm & Node.js Working in Git Bash on Windows
description: A short blog article to make it easy to fix that issue 
date: May 1st, 2025
---


## Getting fnm & Node.js Working in Git Bash on Windows

Managing Node.js versions on Windows with [fnm (Fast Node Manager)](https://github.com/Schniz/fnm) is super convenient—but if you use Git Bash, you’ll hit a couple of surprises. First, Bash doesn’t see the Windows Package Manager (`winget`) out of the box; then, even after installing fnm, you may still get “`node: command not found`.” Finally, VS Code’s integrated Bash won’t load your fnm setup unless you tell it to. Below you’ll find step-by-step fixes—plus explanations of why each tweak is necessary.

---


# Setting up fnm with Git Bash on Windows

## Introduction

Managing multiple Node.js versions on your Windows machine can be a breeze with fnm (Fast Node Manager), but if you're using Git Bash as your primary shell, there are a few extra steps to get everything working smoothly. In this post, I'll walk you through the complete setup process for installing Node.js via fnm on Windows. Along the way, I'll highlight the two key issues I encountered—winget not being recognized in Bash, and VS Code's integrated Bash terminal not picking up fnm or Node—and show you exactly how to resolve them.

## Prerequisites

Before diving in, make sure you have:

- Windows 10 or 11 with an up-to-date installation
- Git for Windows (Git Bash) installed
- Windows Package Manager (winget) available
- Administrative rights (to modify your PATH)

## 1. Installing fnm via winget

The simplest way to install fnm on Windows is through the Windows Package Manager:

```powershell
winget install Schniz.fnm
```

However, if you're running this in Git Bash, you'll immediately hit our first snag: Bash doesn't know about the winget command.

### 1.1 Fixing "bash: winget: command not found"

By default, Git Bash doesn't include WindowsApps in its PATH, so it can't find winget.exe. Here's how to fix that:

#### Locate the WindowsApps folder

In PowerShell, run:

```powershell
(Get-ChildItem -Path $env:LocalAppData\Microsoft\WindowsApps\Microsoft.DesktopAppInstaller_*_x64*\winget.exe).DirectoryName
```

This will print the full path to the folder containing winget.exe.

#### Add it to your Git Bash PATH

Open (or create) your ~/.bashrc and add:

```bash
# Winget in Git Bash
export PATH="$PATH:/c/Users/YourUserName/AppData/Local/Microsoft/WindowsApps"
```

Replace the path with the one you discovered. Then reload:

```bash
source ~/.bashrc
```

Now you can successfully run `winget install Schniz.fnm` inside Git Bash.

## 2. Verifying fnm and Handling the "node: command not found" Error

After installing fnm, running `fnm install --lts` will download and set up an LTS version of Node.js. But you may still see:

```bash
$ node -v
bash: node: command not found
```

This happens because fnm needs you to initialize its environment hook in your shell startup file.

### 2.1 Adding the fnm environment hook

In ~/.bashrc, add the following snippet near the bottom:

```bash
# fnm configuration
FNM_PATH="$HOME/.fnm/bin"
if [ -d "$FNM_PATH" ]; then
  export PATH="$FNM_PATH:$PATH"
  eval "$(fnm env --use-on-cd --shell bash)"
fi
```

This does two things:

- Prepends fnm's own binary folder to your PATH.
- Hooks into directory changes, auto-switching Node versions when it sees a .node-version file.

Reload your shell:

```bash
source ~/.bashrc
```

Now `node -v` and `npm -v` should work perfectly.

## 3. Making fnm and Node Available in VS Code's Integrated Terminal

Even with Git Bash configured, VS Code's integrated terminal might launch Bash in a non-login, non-interactive mode, skipping your ~/.bashrc.

### 3.1 Launching Bash as Login & Interactive

1. Open Settings (Ctrl + ,)
2. Search for terminal.integrated.profiles.windows
3. Edit (or add) the Git Bash profile:

```jsonc
"terminal.integrated.profiles.windows": {
  "Git Bash": {
    "path": "C:\\Program Files\\Git\\bin\\bash.exe",
    "args": ["-l", "-i"]  // Login & interactive
  }
},
"terminal.integrated.defaultProfile.windows": "Git Bash"
```

The `-l` flag ensures Bash reads your login files (~/.bash_profile or ~/.profile), while `-i` ensures it's interactive (reading ~/.bashrc).

Restart VS Code's terminal.

### 3.2 Verifying in VS Code

Open a new terminal in VS Code and run:

```bash
node -v && npm -v && fnm --version
```

You should see the versions of Node, npm, and fnm printed without errors. If not, double-check that your ~/.bashrc includes the fnm hook and that VS Code is indeed using the "Git Bash" profile with the `-l -i` args.

## Conclusion

By explicitly adding WindowsApps to Git Bash's PATH, initializing fnm in your shell startup file, and configuring VS Code to launch Bash as a login and interactive shell, you can seamlessly install and manage multiple Node.js versions on your Windows machine using fnm. Happy coding!