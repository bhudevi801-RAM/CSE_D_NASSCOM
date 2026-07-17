# Lab 1 Notes — Git Fundamentals

## 1. Installing Git

### 1.1 Ubuntu / Linux

```bash
sudo apt update
sudo apt install git
git --version
```

- `sudo apt update` refreshes the list of available packages.
- `sudo apt install git` installs Git itself.
- `git --version` confirms the install and prints the version number (e.g. `git version 2.43.0`).

### 1.2 Windows

There is no `apt` on Windows, so Git is installed differently:

1. Go to **https://git-scm.com/download/win** — the download starts automatically.
2. Run the installer (`Git-2.x.x-64-bit.exe`).
3. Click through the setup wizard. Defaults are fine; make sure **"Git from the command line and also from 3rd-party software"** stays selected so `git` works in any terminal, not just Git Bash.
4. Finish the install, then open **Git Bash** (installed alongside Git) or **Command Prompt / PowerShell** and check:

```powershell
git --version
```

- **Git Bash** gives a Linux-like shell on Windows — every command in these notes (`mkdir`, `cd`, `ls`, `git ...`) works identically inside it.
- If you use plain **CMD/PowerShell** instead, `git` commands behave the same, but a few Linux-only commands need Windows equivalents (see the table below).

### 1.3 Set your identity (once, any OS)

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

This tags every commit you make with your name/email. Git will refuse your first commit without it.

---

## 2. Basic Terminal Commands

| Task | Ubuntu / Git Bash | Windows CMD |
|---|---|---|
| Create a folder | `mkdir foldername` | `mkdir foldername` |
| Enter a folder | `cd foldername` | `cd foldername` |
| List files | `ls` | `dir` |
| Show file contents | `cat filename` | `type filename` |
| Current path | `pwd` | `cd` (no arguments) |

---

## 3. Core Git Workflow

### 3.1 `git init`
```bash
git init
```
Run **once per project**. Creates a hidden `.git` folder that turns a normal folder into a Git repository.

### 3.2 `git status`
```bash
git status
```
Shows the state of every file:
- **Untracked** — new file, Git doesn't know about it yet
- **Tracked** — already added/committed at least once
- **Modified** — changed since the last commit

### 3.3 Staging files
```bash
git add filename     # stage one specific file
git add .            # stage everything in the current folder
```

### 3.4 Committing
```bash
git commit -m "your message"     # commit with an inline message
git commit                        # opens nano editor instead
```
In nano: type your message, then **Ctrl+O → Enter** to save, **Ctrl+X** to exit.

```bash
git commit -a -m "message"
```
Automatically stages **already-tracked** files. It will **not** pick up brand-new (untracked) files — `git add` is still needed the first time a file is created.

### 3.5 Undoing changes
```bash
git restore filename
```
Reverts a file back to its last committed state, discarding uncommitted edits.

### 3.6 Branch naming
```bash
git branch -m main
```
Renames the current branch to `main` — useful right after `git init`, since older Git versions default to `master`.

### 3.7 History and diffs
```bash
git log        # commit history: author, date, message
git diff       # exact line-by-line changes, before committing
```
`git diff` is the best debugging habit — always check it before you commit, so you know exactly what you're about to save.

### 3.8 Removing / renaming tracked files
```bash
git rm filename
git mv oldname newname
```

### 3.9 `.gitignore`
Tells Git which files to never track — usually large or generated files that don't belong in version control.

```
*.pdf
*.jpeg
*.jpg
```

Git *can* track binary files, but we typically ignore ones that are large, generated, or not meaningful to review as text.

---

## Quick Reference

| Command | Purpose |
|---|---|
| `git init` | Start a new repo |
| `git status` | Check file states |
| `git add <file>` / `git add .` | Stage changes |
| `git commit -m "msg"` | Save staged changes |
| `git commit -a -m "msg"` | Stage tracked files + commit in one step |
| `git restore <file>` | Discard uncommitted changes |
| `git branch -m main` | Rename current branch |
| `git log` | View commit history |
| `git diff` | View unstaged changes |
| `git rm` / `git mv` | Remove / rename tracked files |
| `.gitignore` | Exclude files from tracking |

Practice exercises for this material are in **Lab1_Practice_Exercises.ipynb**.
