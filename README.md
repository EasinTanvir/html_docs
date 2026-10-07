# OKF Docs Commands: Team Guide

Two Claude Code commands that turn our normal Markdown docs into **OKF** (Open Knowledge Format) files, and let a teammate sign them off after review.

| Command | Who runs it | What it does |
|---|---|---|
| `/okf-docs` | The **author** of the doc | Adds OKF frontmatter and marks the doc as a **draft** |
| `/okf-verify` | A **reviewer** (a different person) | Marks the doc as **stable** and records who verified it |

## 1. What is OKF? (the 30-second version)

OKF is a plain Markdown file with a small block of settings at the top, called **frontmatter**. It tells people and AI tools who wrote the doc, whether it is trusted, and what kind of doc it is. Your text stays exactly as you wrote it.

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
owner: human:easin
status: draft
generated: { by: "human:easin", at: "2026-10-07" }
tags: [mail, runbook, operations]
---

# Restart the Mail Service

Use this when the mail queue is stuck.
```

After `/okf-verify jack`, two things change (`status` and a new `verified` block):

```yaml
status: stable
verified:
  - { by: "human:jack", at: "2026-10-07" }
```

### What the fields mean

| Field | Meaning |
|---|---|
| `type` | What kind of doc this is: `Runbook`, `Guide`, `Reference` or `Explanation` (see section 6) |
| `title`, `description` | A name and a one-sentence summary |
| `owner` | The person responsible for the doc (the author) |
| `status` | `draft` = written, not reviewed. `stable` = reviewed and trusted. `deprecated` = retired |
| `generated` | Who produced the content and when |
| `verified` | Who reviewed it and when. It stays empty until a reviewer signs off |
| `tags` | Keywords to help people find the doc |

## 2. One-time setup

Do this once per computer.

### Step 1: Install the plugin

Open Claude Code and run these three commands, one at a time:

```
/plugin marketplace add scaccogatto/okf-skills
/plugin install okf@scaccogatto
/reload-plugins
```

You can install it for yourself (user scope) or for one project (project scope). The commands work with either. If you choose project scope, run the install from inside this project's folder.

### Step 2: Check Python

The commands run a checker written in Python. In a terminal, run:

```
python --version
```

If you see a version number, you are fine. If not, install Python from python.org and tick **"Add Python to PATH"**.

You do not need to install anything else. If the checker needs a library called `pyyaml`, the command installs it for you.

### Step 3: Get the command files

The commands are two files inside the project:

```
.claude/commands/okf-docs.md
.claude/commands/okf-verify.md
```

Pull or copy the project so these files exist, then restart Claude Code in that project folder. Type `/okf-` and you should see both commands in the list.

## 3. How to use `/okf-docs` (author)

### The format

```
/okf-docs <your_name> [folder] [file]
```

| Part | Required? | Default | Example |
|---|---|---|---|
| `<your_name>` | Yes | none | `easin` |
| `[folder]` | No | `docs` | `docs` |
| `[file]` | No | `all` | `overview.md` |

Your name may only contain letters, digits, `.`, `_` and `-`. No spaces.

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
   - adds the frontmatter (`type`, `title`, `description`, `owner`, `status: draft`, `generated`, `tags`);
   - picks the doc type;
   - never changes your text or the file name.
5. **Validation.** A checker confirms the files follow the OKF rules. Claude fixes any problems it caused.
6. **Summary.** You get a list of files converted, files skipped (with the reason), and the checker result.

### Files it skips on purpose

| File | Why |
|---|---|
| `index.md`, `log.md` | Reserved OKF files that must not have frontmatter |
| Files that are not `.md` | Not Markdown |
| Files already verified | Converting again would erase a reviewer's sign-off |
| Files with `status: deprecated` | Would bring a retired doc back to life |
| Empty files | Nothing to convert |
| Files with broken frontmatter (a starting `---` but no closing `---`) | It will not guess. Fix the frontmatter by hand, then run it again |

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
4. **Owner check.** If you are the owner of a file, Claude warns you that reviews should be done by someone else and asks you to confirm for that file. Answer `no` to skip it.
5. **Sign-off.** For each remaining file, Claude sets `status: stable` and adds you to `verified` with today's date. If someone verified it before you, their entry is kept and yours is added after it.
6. **Validation** and **summary**, the same as `/okf-docs`.

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

If the doc changes later, edit it, set `status: draft` by hand, remove the old `verified` entries, and ask for a new review.

## 6. Which doc type will I get?

Claude picks one of four types by reading the content:

| Type | Use for | Example |
|---|---|---|
| `Runbook` | Step-by-step instructions for an incident or task | "Restart the Mail Service" |
| `Guide` | How to achieve a goal | "How to customize the form" |
| `Reference` | Facts to look up: tables, fields, settings | "Mail API Reference" |
| `Explanation` | Why something works the way it does | "Why we use a queue" |

Claude chooses these by judgment, so two people running the command may get different types or tags for the same file. If it picked a type you don't agree with, change the `type:` line by hand.

## 7. Errors and what to do

| Message | Meaning | Fix |
|---|---|---|
| `The 'Open Knowledge Format' plugin is not installed yet` | Plugin missing | Do section 2, step 1 |
| `Author name is required as the first argument` | You typed the command without a name | Add your name: `/okf-docs easin` |
| `Verifier name is required...` | Same, for `/okf-verify` | `/okf-verify jack` |
| Name has invalid characters | Your name has spaces or symbols | Use only letters, digits, `.`, `_`, `-` |
| Too many arguments | More than three values | Put names with spaces in quotes |
| `... does not exist` | Wrong folder or file name | Check the spelling and the path |
| Not a Markdown file | You named a file that is not `.md` | Use a `.md` file |
| Command not in the `/` list | The files are missing or Claude Code is not restarted | Check `.claude/commands/` and restart |
| `No module named 'yaml'` | A checker library is missing | The command installs it itself. If it can't, run `python -m pip install --user pyyaml` |
| Checker reports errors on skipped files | Expected for empty or broken files | Fix the file by hand, then run again |

## 8. Good to know

- **Your text is safe.** The commands only add or change frontmatter. They never edit your body text or rename files.
- **Run it as often as you like.** Running `/okf-docs` again on a draft file is harmless. Verified files are skipped.
- **Always try one file first** if you are nervous: `/okf-docs yourname docs filename.md`.
- **Undo.** If you committed to git first, run `git diff` to see exactly what changed and `git checkout -- <file>` to undo.
- **Tested on Windows.** Mac and Linux should work, but have not been tested yet. If you hit a problem there, tell the team.
- **Today's date** is filled in automatically.

## 9. Quick cheat sheet

```
Author:     /okf-docs  <me>  docs  <file>     # make a draft
Reviewer:   /okf-verify <me> docs  <file>     # approve it
Everything: leave out <file> and answer "yes" when asked
```
