# OKF Docs Commands: Team Guide

Two Claude Code commands that turn our normal Markdown docs into **OKF** (Open Knowledge Format) v0.2 files, and let a teammate sign them off after review.

| Command | Who runs it | What it does |
|---|---|---|
| `/okf-docs` | The **author** of the doc | Adds OKF frontmatter and marks the doc as a **draft** |
| `/okf-verify` | A **reviewer** (a different person) | Marks the doc as **stable** and records who verified it |

The output follows the official OKF v0.2 specification. The plugin ships a copy of it as `skills/okf/reference/SPEC.md`, and the same spec is at [GoogleCloudPlatform/open-knowledge-format](https://github.com/GoogleCloudPlatform/open-knowledge-format). Section numbers such as §5.2 refer to that spec.

## 1. What is OKF? (the 30-second version)

OKF is a plain Markdown file with a small block of settings at the top, called **frontmatter**. It tells people and AI tools what kind of doc it is, who wrote it, and whether it has been reviewed. Your text stays exactly as you wrote it.

Before:

```markdown
# Restart the Mail Service

Use this when the mail queue is stuck.
```

After `/okf-docs easin`:

```markdown
---
type: Runbook
title: "Restart the Mail Service"
description: "Steps to restart the mail service when the mail queue is stuck."
tags: [mail, operations]
status: draft
generated: { by: human:easin, at: 2026-10-07T09:15:00Z }
---

# Restart the Mail Service

Use this when the mail queue is stuck.
```

After `/okf-verify jack`, two things change: `status` becomes `stable`, and a `verified` list is added under `generated`:

```yaml
status: stable
generated: { by: human:easin, at: 2026-10-07T09:15:00Z }
verified:
  - { by: human:jack, at: 2026-10-08T14:02:11Z }
```

Every file gets the same keys in the same order and the same style, whoever runs the command.

### What the fields mean

| Field | Meaning | Spec |
|---|---|---|
| `type` | What kind of doc this is: `Runbook`, `Guide`, `Reference` or `Explanation` (see section 6). The only required field | §4.1 |
| `title`, `description` | A name and a one-sentence summary | §4.1 |
| `tags` | Keywords to help people find the doc | §4.1 |
| `status` | `draft` = written, not reviewed. `stable` = reviewed and ready to use. `deprecated` = retired. No other value is valid | §5.4 |
| `generated` | Who wrote the content (`human:<name>`) and when, in UTC | §5.2, §7 |
| `verified` | Who reviewed it and when. Missing until a reviewer signs off. Several reviewers form a list | §5.2, §5.3 |

Times are full UTC timestamps such as `2026-10-07T09:15:00Z`, because the spec requires every time value to be an ISO 8601 datetime with a UTC offset (§5). The commands read the time from your computer's clock.

## 2. Prerequisites and one-time setup

Do these steps **in order**, once per computer. Each step has a check, so you know it worked before moving on. They work on Windows, macOS and Linux.

### What you need (summary)

| # | What | Required? | Why |
|---|---|---|---|
| 1 | **Claude Code** (signed in) | Yes | The commands run inside it |
| 2 | **`uv`** *or* **Python 3.11+ with PyYAML** | Yes, one of the two | The OKF checker is a Python script. `uv` is the easiest: it fetches the right Python and PyYAML by itself |
| 3 | **The OKF plugin** (`okf@scaccogatto`) | Yes | Provides the OKF skills and the checker |
| 4 | **This repository** (Git) | Yes | Contains the two commands in `.claude/commands/` |

### Step 1: Install Claude Code

Install Claude Code and sign in. To check, open a terminal and run:

```
claude --version
```

You should see a version number.

### Step 2: Install `uv` (recommended)

`uv` runs the OKF checker with the right Python version and PyYAML, so you do not have to manage Python yourself. The OKF plugin also needs it for its optional `bundle` server.

- **Windows** (PowerShell): `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`
- **macOS / Linux**: `curl -LsSf https://astral.sh/uv/install.sh | sh`

Close and reopen your terminal, then check:

```
uv --version
```

You should see a version number. If this worked, **skip Step 3**.

### Step 3 (only if you did not install `uv`): Python 3.11+ and PyYAML

Check your Python. Use the first command that prints a version:

```
python3 --version
python --version
py -3 --version
```

It must be **3.11 or higher** (the checker's own requirement). If not, install the latest Python from python.org (on Windows, tick **"Add python.exe to PATH"** in the installer, then reopen the terminal). On Windows, `python3` may print "Python was not found": that is a Microsoft Store shortcut. Ignore it and use `python`.

Install PyYAML with the same command name that worked above (here `python`):

```
python -m pip install --user pyyaml
```

- **macOS (Homebrew) or Linux** may refuse with `externally-managed-environment`. Install `uv` (Step 2) instead, or on Debian/Ubuntu run `sudo apt install python3-yaml`.

Check it worked:

```
python -c "import yaml; print('PyYAML OK')"
```

### Step 4: Install the OKF plugin

Open Claude Code **inside this project's folder** and run these commands, one at a time:

```
/plugin marketplace add scaccogatto/okf-skills
/plugin install okf@scaccogatto
```

When asked for a scope, choose **"Install for you (user scope)"** (simplest) or **"Install for all collaborators on this repository (project scope)"**. Both work. If Claude Code says `Run /reload-plugins to apply`, run:

```
/reload-plugins
```

Prefer the terminal? These do the same, from your shell:

```
claude plugin marketplace add scaccogatto/okf-skills
claude plugin install okf@scaccogatto
```

This repository already lists the plugin in `.claude/settings.json`, so Claude Code reminds you if it is not installed yet. That file only *enables* the plugin; each person still has to install it once with the commands above.

Check it worked: type `/okf` in Claude Code. You should see `okf:okf` and `okf:validate` in the list. You can also run `claude plugin list` in your shell.

### Step 5: Get the command files

Clone or pull this repository. The commands are these two files:

```
.claude/commands/okf-docs.md
.claude/commands/okf-verify.md
```

Start Claude Code **from the project folder** (or restart it after pulling). The first time, Claude Code asks whether you trust the folder: answer yes, otherwise it ignores the project settings.

Check it worked: type `/okf-` and you should see both `okf-docs` and `okf-verify`.

### Final check

Before your first real run, make sure you can answer "yes" to all of these:

- [ ] `claude --version` prints a version
- [ ] `uv --version` prints a version, **or** Python 3.11+ is installed and `python -c "import yaml"` prints no error
- [ ] `/okf` shows `okf:okf` and `okf:validate`
- [ ] `/okf-` shows `okf-docs` and `okf-verify`

If one is "no", go back to that step. Section 7 lists common errors.

## 3. How to use `/okf-docs` (author)

### The format

```
/okf-docs <your_name> [folder] [file]
```

| Part | Required? | Default | Example |
|---|---|---|---|
| `<your_name>` | Yes | none | `easin` |
| `[folder]` | No | `docs` | `docs` |
| `[file]` | No | `all` | `overview.md` or `sub/guide.md` |

Your name may only contain letters, digits, `.`, `_` and `-`. No spaces. **Always use the same spelling** (the team convention is lowercase, for example `easin`), because the reviewer check compares names.

### Examples

Convert **one file** (no question asked):

```
/okf-docs easin docs overview.md
```

Convert **every file** in the `docs` folder (Claude asks you to confirm):

```
/okf-docs easin
```

Convert every file in a different folder:

```
/okf-docs easin guides
```

A file name with spaces goes in quotes:

```
/okf-docs easin docs "my notes.md"
```

### What happens, step by step

1. **Plugin check.** If the OKF plugin is missing, you get an error with the install steps.
2. **Argument check.** It reads your name, folder and file. It stops with a clear error if something is wrong.
3. **Confirmation (only for "all").** Claude asks: *"Are you sure you want to proceed? (yes/no)"*. Type `yes` to continue. Anything else cancels and changes nothing.
4. **Conversion.** For each file, Claude:
   - adds the frontmatter (`type`, `title`, `description`, `tags`, `status: draft`, `generated`) in the fixed order shown in section 1;
   - picks the doc type;
   - keeps any frontmatter keys the file already had;
   - never changes your text or the file name.
5. **Validation.** The plugin's checker confirms the files follow the OKF rules (`--strict` for a folder). Claude fixes any frontmatter problems it caused.
6. **Summary.** You get a list of files converted, files skipped (with the reason), and the checker result.

### Files it skips on purpose

| File | Why |
|---|---|
| `index.md`, `log.md` | Reserved OKF files, not documents (§8, §9) |
| Files that are not `.md` | Not Markdown |
| Files already verified | Converting again would erase a reviewer's sign-off |
| Files with `status: deprecated` | Would bring a retired doc back to life |
| Empty files | Nothing to convert |
| Files with broken frontmatter (a starting `---` but no closing `---`, or invalid YAML) | It will not guess. Fix the frontmatter by hand, then run it again |

### Old OKF v0.1 files

If a file still has the old `timestamp:` field or a `# Citations` list, `/okf-docs` upgrades it the same way the plugin's `--migrate` does (§13.1): `timestamp` becomes `generated.at` (the original date is kept), and the links in `# Citations` move into a `sources` list in the frontmatter. That `# Citations` section is the **only** body text the commands ever move.

## 4. How to use `/okf-verify` (reviewer)

Verification means: *"I read this doc and I trust it."* **Do not verify your own doc.** A second person should review it.

### The format

```
/okf-verify <your_name> [folder] [file]
```

It uses the same three parts as `/okf-docs`. `<your_name>` is the **reviewer's** name.

### Examples

Verify **one file**:

```
/okf-verify jack docs overview.md
```

Verify **every file** in `docs` (Claude asks you to confirm):

```
/okf-verify jack
```

### What happens, step by step

1. **Plugin check** and **argument check**, the same as `/okf-docs`.
2. **Confirmation** (only for "all"). Answer `yes` to continue.
3. **Skip check.** Files without OKF frontmatter, empty files, deprecated files, and files you have already verified are skipped and listed.
4. **Author check.** If you wrote the file (your name is in `generated`), Claude warns you that reviews should be done by someone else and asks you to confirm for that file. Answer `no` to skip it.
5. **Sign-off.** For each remaining file, Claude sets `status: stable` and adds you to `verified` with the current UTC time. If someone verified it before you, their entry is kept and yours is added after it. Nothing else in the file changes.
6. **Validation** and **summary**, the same as `/okf-docs`. If the checker finds a problem that is not about the sign-off (for example a missing `title`), it is reported, not fixed: run `/okf-docs` or fix it by hand.

## 5. The normal team workflow

```
Author writes a doc in Markdown
        |
        v
Author runs   /okf-docs <author> docs <file>     -> status: draft
        |
        v
Author commits the file and opens a pull request
        |
        v
Reviewer reads the doc in the pull request
        |
        v
Reviewer runs  /okf-verify <reviewer> docs <file> -> status: stable
        |
        v
Reviewer commits the sign-off, and the PR is merged
```

If the doc changes later, edit it, then by hand set `status: draft`, update `generated.at` to the current UTC time, remove the old `verified` entries, and ask for a new review.

Always commit before running a command on many files, so you can review the result with `git diff`.

## 6. Which doc type will I get?

Claude picks one of four types by reading the content. The spec lets teams choose their own type names (§4.1); we use these four:

| Type | Use for | Example |
|---|---|---|
| `Runbook` | Step-by-step instructions for an incident or task | "Restart the Mail Service" |
| `Guide` | How to achieve a goal | "How to customize the form" |
| `Reference` | Facts to look up: tables, fields, settings | "Mail API Reference" |
| `Explanation` | Why something works the way it does | "Why we use a queue" |

Claude chooses the type, description and tags by judgment, so two people may get slightly different values for the same file. The keys, their order, the format and the status are always the same. If you disagree with the type, change the `type:` line by hand.

## 7. Errors and what to do

| Message | Meaning | Fix |
|---|---|---|
| `The 'Open Knowledge Format' plugin is not installed yet` | Plugin missing | Section 2, Step 4 |
| `Author name is required as the first argument` | You typed the command without a name | Add your name: `/okf-docs easin` |
| `Verifier name is required...` | Same, for `/okf-verify` | `/okf-verify jack` |
| Name has invalid characters | Your name has spaces or symbols | Use only letters, digits, `.`, `_`, `-` |
| Too many arguments | More than three values | Put file names with spaces in quotes |
| `... does not exist` | Wrong folder or file name | Check the spelling and the path (relative to the project folder) |
| Not a Markdown file | You named a file that is not `.md` | Use a `.md` file |
| Command not in the `/` list | The files are missing, or Claude Code was started outside the project folder | Pull the repo, `cd` into it, start Claude Code again |
| `No module named 'yaml'` | No `uv`, and PyYAML is missing | Section 2, Step 2 (install `uv`) or Step 3 |
| `Python was not found` (Windows) | The Microsoft Store `python3` shortcut | Harmless: the commands try `uv`, `python` and `py` next |
| `plugin:okf:bundle` failed to connect | The plugin's optional server needs `uv` | Install `uv` (Section 2, Step 2). The two commands work without it |
| Checker reports errors on skipped files | Expected for empty or broken files | Fix the file by hand, then run again |

## 8. Good to know

- **Your text is safe.** The commands only add or change frontmatter. They never edit your body text or rename files (the one exception is the old v0.1 `# Citations` section, see section 3).
- **Run it as often as you like.** Running `/okf-docs` again on a draft file only fills in missing fields and keeps the original `generated` entry. Verified files are skipped.
- **Always try one file first** if you are nervous: `/okf-docs yourname docs filename.md`.
- **Undo.** If you committed to git first, run `git diff` to see exactly what changed and `git checkout -- <file>` to undo.
- **Line endings.** The repository's `.gitattributes` stores `.md` files with LF endings, so Windows and Mac users get identical files and clean diffs.
- **Non-interactive runs** (`claude -p`): `all` needs a "yes" answer, so a one-shot run stops at the question and changes nothing. Name a single file instead.
- **Times** are filled in automatically from your computer's clock, in UTC.

## 9. Quick cheat sheet

```
Author:     /okf-docs  <me>  docs  <file>     # make a draft
Reviewer:   /okf-verify <me> docs  <file>     # approve it
Everything: leave out <file> and answer "yes" when asked
```
