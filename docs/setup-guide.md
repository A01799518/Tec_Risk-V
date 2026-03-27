# Quick Start Guide: Setup & Workflow
**Project:** RISC-V CPU (Verilog/RTL)

> This guide walks you through setting up your computer to design hardware the way it's done in the industry, connecting Windows, Linux (Ubuntu), and GitHub through VS Code.
>
> If you're already a Linux and Git expert, you probably don't need this. If the phrase "clone a repository" still sounds like something from biology class, keep reading.

---

## Table of Contents

1. [PHASE 1 — The Tools (VS Code Extensions)](#phase-1--the-tools-vs-code-extensions)
2. [PHASE 2 — Entering the Matrix (WSL Setup)](#phase-2--entering-the-matrix-wsl-setup)
3. [PHASE 3 — The Workflow (Git for Mortals)](#phase-3--the-workflow-git-for-mortals)
4. [Common Mistakes and How to Survive Them](#common-mistakes-and-how-to-survive-them)

---

## PHASE 1 — The Tools (VS Code Extensions)

We shall now add "superpowers" to our terminal. Install these extensions from the Extensions tab (`Ctrl+Shift+X`):

### Required Extensions

| Extension | Author | What it does |
|-----------|--------|--------------|
| **WSL** | Microsoft | Connects VS Code to the Ubuntu you installed on Windows. Without this, you cannot run anything at all. |
| **Verilog-HDL/SystemVerilog** | mshr-h | Autocompletes code, syntax highlights, and flags RTL errors *in real time* (built-in Linter). |
| **Tcl Language Support** | bitwisecook | Synopsys synthesis scripts are written in Tcl. This makes them readable instead of looking like ancient runes. |

### Recommended Extensions

| Extension | What it does |
|-----------|--------------|
| **Git Graph** | Shows a visual tree of all repository branches. Extremely useful for understanding who is doing what without ever opening the terminal. |
| **GitLens** | Adds Git information directly inline on every line of code. Great for knowing who wrote what and when. |

> **Note:** Install extensions *inside* WSL too, not just on the Windows side. VS Code will prompt you to do this automatically the first time you open it from Ubuntu. Say yes.

---

## PHASE 2 — WSL Setup

> **Golden rule:** Never work on your `.v` files directly from the Windows file explorer. Always do it "from inside" Ubuntu. If you're not sure why — trust the process. You'll understand the first time something mysteriously fails to compile.

### Step by Step

**1. Open the Ubuntu terminal**

Search for "Ubuntu" in the Windows Start Menu and open it. You'll see a terminal with a prompt like this:

```
yourname@LAPTOP-XXXXX:~$
```

Welcome to Linux. It's not as scary as it looks.

**2. Navigate to where you want to clone the project**

Your Windows files live inside `/mnt/c/`. For example:

```bash
cd /mnt/c/Users/YourUsername/Documents/
```

Replace `YourUsername` with your actual Windows username. You can confirm it by opening the Windows File Explorer and checking your user folder name.

**3. Clone the repository**

```bash
git clone https://github.com/A01799518/Tec_RISC-V.git
```

This downloads the entire project into a folder called `Tec_RISC-V`.

**4. Enter the project folder**

```bash
cd Tec_RISC-V
```

**5. Open VS Code connected to Linux**

```bash
code .
```

The dot (`.`) means "open VS Code *here*, in this folder". VS Code will launch automatically connected to your Ubuntu environment.

### How do I know it worked?

In the bottom-left corner of VS Code, you should see a green badge that reads:

```
><  WSL: Ubuntu
```

If you see that, you did it right. You're in. If you see something different, close VS Code and repeat Step 5 from the Ubuntu terminal.

---

## PHASE 3 — The Workflow (Git for Mortals)

This is the cycle we will repeat **every single time** we work on the project. 

> **Reminder from the Git Convention Guide:** Never work directly on `main`. Every module or fix lives on its own branch. Every commit follows the format `<type>(<scope>): <description>`. If you don't remember the details, check `git-convention.md`.

Open the integrated VS Code terminal with `Ctrl + ~` and follow these steps:

---

### Step 1 — Sync up before you start

Before writing a single line of code, make sure you have the latest version of the project:

```bash
git checkout main
git pull origin main
```

This prevents you from working on stale code and then having to untangle painful merge conflicts later. Do it every time. No exceptions.

---

### Step 2 — Create your own branch

Create an isolated space where you can do your work without affecting everyone else. The branch name must follow the format from the [Git Convention Guide](./git-convention.md):

```
<type>/<scope>-<brief-description>
```

```bash
git checkout -b feat/alu-combinational-logic
```

More valid examples:

```bash
git checkout -b feat/fetch-pc-increment
git checkout -b fix/decode-imm-signext
git checkout -b sim/alu-directed-tests
```

> ⚠️ **Invalid branch names** (do not use these):
> ```
> alu-draft          # missing the type prefix (feat/, fix/, etc.)
> my-branch          # says nothing about what it is or which module it belongs to
> feature_ALU        # uses underscores and uppercase — both incorrect
> ```

---

### Step 3 — Save your changes (Add & Commit)

You wrote your RTL and want to save it to your local history. Before running `add`, **check which files you are about to include:**

```bash
git status
```

Read the list carefully. If you see `.vcd`, `.log`, or `/work` directories — **do not include them**. They should already be covered by `.gitignore`, but it's always good to confirm. Then add only your source files:

```bash
# Option A: Add a specific file (safer)
git add rtl/alu.v

# Option B: Add everything (only use this if git status looks clean)
git add .
```

Now commit following the format from the [Git Convention Guide](./git-convention.md):

```bash
git commit -m "feat(alu): add combinational logic for arithmetic operations"
```

> ⚠️ **Invalid commits** (do not use these):
> ```bash
> git commit -m "added the ALU logic"     # does not follow the format
> git commit -m "fix"                     # way too vague
> git commit -m "WIP"                     # never push Work In Progress to a shared branch
> ```

---

### Step 4 — Upload to GitHub (Push)

Send your branch to GitHub so everyone else can see it:

```bash
git push origin feat/alu-combinational-logic
```

Use the **exact same branch name** you created in Step 2.

---

### Step 5 — Open a Pull Request (PR)

1. Go to the repository page on GitHub: `https://github.com/A01799518/Tec_RISC-V`
2. A yellow banner will appear saying **"Compare & pull request"** — click it.
3. Write a short description of what you did and why.
4. Let the team know you have pushed your part.
5. Wait for at least **one teammate to review** using the [Code Review Checklist](./git-convention.md#5-code-review-checklist) from the Convention Guide.
6. Once approved, click **Merge** to bring it into `main`.
7. **Delete your branch** after the merge. GitHub will offer you a button to do this automatically.

---

## Common Mistakes and How to Survive Them

### "It says there are merge conflicts"

This happens when two people modified the same file. Do not panic.

```bash
# First, bring the changes from main into your branch
git checkout your-branch
git merge main
```

Git will mark the conflicts inside the file with `<<<<<<< HEAD`. Open the file in VS Code, decide which version is correct, delete the conflict markers, save, and make a new commit. If you're not sure which version to keep, talk to whoever made the other change — don't guess.

---

### "I accidentally pushed a .vcd file"

First, breathe. Then:

```bash
# Remove the file from Git tracking (without deleting it from your disk)
git rm --cached your_file.vcd

# Add the extension to .gitignore if it wasn't there already
echo "*.vcd" >> .gitignore

# Make a commit that fixes the mistake
git add .gitignore
git commit -m "chore: remove vcd from tracking and update gitignore"
git push origin your-branch
```

If it already reached `main`, let the team know — cleaning the `main` history requires an extra step that should not be done alone.

---

### "I want to undo my last commit"

If you have **not** pushed it yet:

```bash
# Undoes the commit but keeps your file changes intact
git reset --soft HEAD~1
```

If you **already** pushed it, do not use `reset` on a shared branch. Talk to the team first.

---

### "I forgot to create a branch and worked directly on main"

```bash
# Save your work to a new branch (nothing is lost)
git checkout -b feat/alu-combinational-logic

# Restore main to its original clean state
git checkout main
git reset --hard origin/main
```

Your work is safe on the new branch. `main` is clean again.

---

