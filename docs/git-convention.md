# RISC CPU — Git Convention Guide
**Project:** RISC-Based CPU (Verilog/RTL)
**Prepared for:** Hardware Design Team
**Document Version:** 1.0

> This guide defines the Git workflow, commit standards, branching strategy, hardware-specific rules, and code review checklist for the RISC CPU project.
> All contributors are expected to follow these conventions without exception. If you believe a rule needs to change, open a discussion — do not silently deviate.

---

## Table of Contents

1. [Conventional Commit Messages](#1-conventional-commit-messages)
2. [Branching Strategy](#2-branching-strategy)
3. [Hardware-Specific Rules](#3-hardware-specific-rules)
4. [.gitignore Reference](#4-gitignore-reference)
5. [Code Review Checklist](#5-code-review-checklist)
6. [Quick Reference Card](#6-quick-reference-card)

---

## 1. Conventional Commit Messages

Every commit must follow this format exactly:

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

### 1.1 Types

| Type    | When to use |
|---------|-------------|
| `feat`  | Adding new RTL functionality (a new module, a new instruction support) |
| `fix`   | Correcting a bug in RTL or simulation logic |
| `rtl`   | Refactoring, renaming, or restructuring existing RTL without changing behavior |
| `sim`   | Changes to testbenches, simulation scripts, or waveform dumps |
| `syn`   | Synthesis scripts, constraints, or timing-related changes (Synopsys/DC/Vivado) |
| `docs`  | Documentation updates (READMEs, comments, this guide) |
| `ci`    | Changes to CI scripts, Makefiles, or automation |
| `chore` | Housekeeping (updating .gitignore, renaming files, reorganizing directories) |

### 1.2 Scopes

The scope identifies which hardware module or area of the project is affected. Use the exact module name as defined in your RTL hierarchy. Examples:

| Scope       | What it refers to |
|-------------|-------------------|
| `alu`       | Arithmetic Logic Unit |
| `decode`    | Instruction Decode stage |
| `fetch`     | Instruction Fetch stage |
| `execute`   | Execute stage |
| `writeback` | Write-Back stage |
| `regfile`   | Register File |
| `mem`       | Memory interface or data memory |
| `top`       | Top-level integration module |
| `tb`        | Testbench (general, when not tied to one module) |
| `ctrl`      | Control unit or FSM |
| `hazard`    | Hazard detection and forwarding unit |

If your change spans multiple scopes, use the most significant one or use `top`.

### 1.3 Short Description Rules

- Use the **imperative mood**: "add", "fix", "remove" — not "added", "fixing", "removed"
- Keep it under **72 characters**
- Do **not** end with a period
- Be specific — avoid vague descriptions like "update stuff" or "fix bug"

### 1.4 Examples

```
feat(alu): add support for signed multiplication (MULH)

fix(decode): correct opcode mask for I-type immediate sign extension

rtl(regfile): rename internal signals to match naming convention

sim(tb): add directed test for load-use hazard stall

syn(top): add false path constraints for reset synchronizer

docs(fetch): document PC increment logic in module header

chore: update .gitignore to exclude Synopsys work libraries
```

### 1.5 Commit Body (Optional but Encouraged)

For non-trivial changes, include a body that explains **why**, not just what:

```
fix(hazard): prevent forwarding from load-use slot on RAW hazard

The forwarding path was incorrectly passing the pre-memory value
when the preceding instruction was a load (LW). This caused a
data corruption on back-to-back load-to-use sequences.
Stall logic now inserts a bubble when a load-use hazard is detected.
```

---

## 2. Branching Strategy

The project uses a **two-tier branching model**: a stable main branch and short-lived feature branches.

### 2.1 Branch Overview

```
main
 └── feat/fetch-pc-increment
 └── feat/alu-mul-extension
 └── fix/decode-imm-signext
 └── sim/hazard-directed-tests
 └── syn/dc-timing-constraints
```

### 2.2 The `main` Branch

- Represents the **latest stable, working version** of the CPU.
- **Direct pushes to `main` are forbidden.** All changes must arrive via a Pull Request (PR).
- Every commit on `main` must have passed simulation and, if applicable, a synthesis check.
- The `main` branch is tagged at design milestones (see Section 2.5).

### 2.3 Feature Branches

Create a feature branch whenever you begin work on a module, fix, or simulation task. The branch name must follow this format:

```
<type>/<scope>-<brief-description>
```

Use hyphens (`-`) between words. No underscores, no uppercase letters.

**Examples:**

```
feat/fetch-pc-increment
feat/alu-mul-extension
fix/decode-imm-signext
fix/regfile-read-during-write
sim/alu-exhaustive-tb
sim/pipeline-stall-coverage
syn/dc-compile-script
rtl/ctrl-fsm-refactor
docs/readme-pipeline-diagram
```

### 2.4 Branch Lifecycle

1. **Create** the branch from the latest `main`:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b feat/alu-mul-extension
   ```

2. **Work** on your feature with small, focused commits.

3. **Push** your branch and open a Pull Request targeting `main`.

4. **Review** — at least one other team member must approve (see Section 5).

5. **Merge** via **Squash and Merge** to keep `main` history clean, unless the full commit history is meaningful.

6. **Delete** the feature branch after merge. Do not leave stale branches.

### 2.5 Version Tags

Tag `main` at every significant milestone using annotated tags:

```bash
git tag -a v0.1.0 -m "Single-cycle datapath complete, ALU and Regfile verified"
git push origin v0.1.0
```

Suggested versioning: `v<major>.<minor>.<patch>`

| Version | Meaning |
|---------|---------|
| `v0.1.x` | Single-cycle datapath |
| `v0.2.x` | Pipelined datapath (no hazard handling) |
| `v0.3.x` | Hazard detection and forwarding |
| `v1.0.0` | Full ISA support, verified, synthesis-clean |

---

## 3. Hardware-Specific Rules

RTL projects generate large binary and temporary files that must **never** enter the repository. This section defines what stays out and why.

### 3.1 Files That Must Never Be Committed

#### Simulation Output Files

| File Type | Extension | Reason |
|-----------|-----------|--------|
| Value Change Dump | `.vcd` | Binary waveform files; can exceed hundreds of MB per simulation run |
| FSDB Waveform | `.fsdb` | Verdi/Novas proprietary format; extremely large |
| Simulation log | `.log` | Generated automatically; not source-controlled content |
| Compiled simulation | `simv`, `*.simv` | Binary executables; environment-specific |
| VCS/ModelSim libraries | `work/`, `csrc/`, `simv.daidir/` | Compiled model directories; entirely regenerable |

#### Synthesis Temporary Files

| Directory/File | Tool | Reason |
|----------------|------|--------|
| `work/` | Synopsys DC, Genus | Compiled design database; regenerable |
| `logs/` | Any EDA tool | Tool-generated logs; can be large and noisy |
| `reports/` (auto-generated) | DC, PrimeTime | Regenerable from scripts |
| `*.syn` scratch files | Various | Temporary synthesis state |
| `.snps_` prefixed files | Synopsys | Internal tool state |
| `default.svf` | Synopsys | Security file; not portable |

#### Other Hardware Tool Artifacts

| File | Tool |
|------|------|
| `*.jou` | Vivado journal |
| `*.str` | Vivado strategy |
| `.Xil/` | Vivado temp directory |
| `*.runs/`, `*.cache/`, `*.ip_user_files/` | Vivado project directories |
| `*.mem` (compiled) | Tool-generated memory initialization |
| `transcript` | QuestaSim/ModelSim |

### 3.2 Files That Should Be Committed

| File Type | Extension | Notes |
|-----------|-----------|-------|
| RTL source | `.v`, `.sv` | All synthesizable design files |
| Testbenches | `_tb.v`, `_tb.sv` | Simulation verification source |
| Constraints | `.sdc`, `.xdc` | Timing and physical constraints |
| Simulation scripts | `.tcl`, `.do`, `Makefile` | Reproducible run scripts |
| Synthesis scripts | `.tcl` | DC/Vivado build scripts |
| Memory initialization | `.hex`, `.mif` | Static source, not generated |
| Documentation | `.md`, `.pdf` | Design docs, datasheets |
| Waveform config | `.do` (display only) | Waveform format files, not dumps |

### 3.3 Handling Simulation Results

Simulation results are **ephemeral** by nature. The goal is reproducibility, not archiving outputs.

- Commit the **script** that runs the simulation, not the output.
- If a waveform needs to be shared for debugging, use a temporary file-sharing link — not the repository.
- Pass/fail results should be captured by a short log summary in your PR description, not by uploading log files.

---

## 4. `.gitignore` Reference

Place this `.gitignore` at the root of the repository. Review it at the start of any new tool integration.

```gitignore
# ============================================================
# RISC CPU Project — .gitignore
# ============================================================

# --- Simulation Output ---
*.vcd
*.fsdb
*.vpd
*.shm/
*.trn
*.dsn
*.log
simv
simv.daidir/
csrc/
ucli.key

# --- ModelSim / QuestaSim ---
work/
transcript
vsim.wlf
*.wlf

# --- Synopsys Design Compiler ---
work/
logs/
.synopsys_dc.setup
default.svf
command.log
*.syn
*.pvl
*.mr
*.milkyway/

# --- Vivado ---
*.jou
*.str
.Xil/
*.runs/
*.cache/
*.ip_user_files/
*.gen/
*.hw/
*.sim/
*.srcs/
*.xpr

# --- Genus / Innovus ---
genus.cmd*
innovus.cmd*
*.enc.dat
fcheck.enc

# --- OS and Editor Artifacts ---
.DS_Store
Thumbs.db
*.swp
*.swo
*~
.vscode/
.idea/

# --- Python (if used for scripts) ---
__pycache__/
*.pyc
*.pyo
```

> **Rule:** If you find yourself about to `git add` a file that is not source code, a script, a constraint, or documentation — stop and verify it should not be in `.gitignore` first.

---

## 5. Code Review Checklist

Every Pull Request must be reviewed against this checklist before merging to `main`. The **author** prepares the PR; the **reviewer** verifies each item.

### 5.1 Author Checklist (Before Opening a PR)

**RTL Quality**
- [ ] All module ports are named consistently with the project naming convention
- [ ] No hardcoded parameters — use `parameter` or `localparam`
- [ ] Reset behavior is synchronous (or asynchronous, as decided for the project) and consistent across all flip-flops
- [ ] No latches inferred (verify with synthesis or linter warnings)
- [ ] No unused signals left in the design (`wire` declarations without connections)
- [ ] All `always` blocks have a complete sensitivity list (use `always @(*)` or `always_comb` for combinational)

**Simulation**
- [ ] At least one directed testbench is included or updated for the changed module
- [ ] The testbench covers both normal operation and at least one edge case
- [ ] Simulation runs to completion without `$fatal` or assertion failures
- [ ] No X-propagation issues on critical paths (check waveform or use `$display` assertions)

**Synthesis (if applicable)**
- [ ] Module compiles without errors in the target tool (DC, Vivado, or equivalent)
- [ ] No critical warnings about multi-driven nets or unresolved references
- [ ] Timing constraints are met or violations are documented in the PR description

**Repository Hygiene**
- [ ] No `.vcd`, `.fsdb`, `.log`, or tool-generated files staged for commit (`git status` is clean except for source files)
- [ ] `.gitignore` covers any new tool artifacts introduced
- [ ] Branch is up to date with `main` (rebase or merge before opening PR)
- [ ] Commit messages follow the Conventional Commits format (Section 1)

### 5.2 Reviewer Checklist (Before Approving)

**Understanding**
- [ ] The PR description clearly explains **what** was changed and **why**
- [ ] The scope of the change matches the branch name and commit messages

**RTL Review**
- [ ] Logic is correct for the intended operation (trace through the critical paths manually if needed)
- [ ] No unintended behavioral changes to modules outside the PR scope
- [ ] Synthesis-unfriendly constructs are absent (e.g., delays `#10` in RTL, `initial` blocks in synthesizable code)

**Simulation Evidence**
- [ ] PR includes a summary of simulation results (pass/fail, test cases covered)
- [ ] Testbench adequately covers the new or changed logic

**Documentation**
- [ ] Module headers and inline comments are updated if the interface or behavior changed
- [ ] Any new parameters or ports are documented

**Final Gate**
- [ ] All author checklist items are confirmed complete
- [ ] No open questions or unresolved review comments remain
- [ ] Approval is given — or changes are requested with specific, actionable feedback

---

## 6. Quick Reference Card

```
COMMIT FORMAT
─────────────────────────────────────────────────
<type>(<scope>): <imperative description>

Types:   feat | fix | rtl | sim | syn | docs | ci | chore
Scopes:  alu | decode | fetch | execute | writeback |
         regfile | mem | ctrl | hazard | top | tb

BRANCH FORMAT
─────────────────────────────────────────────────
<type>/<scope>-<brief-description>

Examples:
  feat/alu-mul-extension
  fix/decode-imm-signext
  sim/pipeline-stall-coverage

GOLDEN RULES
─────────────────────────────────────────────────
1.  Never push directly to main
2.  Never commit .vcd, .fsdb, .log, or /work
3.  One PR = one logical change
4.  Simulation must pass before review
5.  Delete your branch after merge
6.  When in doubt, check .gitignore first
```

---

*For questions about this guide, open an issue or contact the project lead. Do not silently modify these conventions — consistency is the point.*