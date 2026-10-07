---
description: Remove the verification from OKF documents and set them back to draft, so they can be edited and reviewed again.
argument-hint: [directory] [file|all]
disable-model-invocation: true
---

Raw arguments: `$ARGUMENTS`

Perform the following steps carefully. Parse the raw arguments above yourself, as described in Step 2 (do not rely on numbered argument placeholders: their numbering differs between Claude Code versions).

The OKF rules below come from the OKF v0.2 specification shipped with the plugin (`skills/okf/reference/SPEC.md`). Section numbers (§) refer to it.

### Step 1: Pre-requisite Check
Check if the **Open Knowledge Format** plugin (`okf@scaccogatto`) is installed. The plugin counts as installed if the `okf:okf` and `okf:validate` skills are available in this session. If you cannot tell, look for `okf@scaccogatto` in `~/.claude/plugins/installed_plugins.json`, but only count an entry whose scope is `user`, or whose `projectPath` is the current project (an entry for another project does not enable the plugin here).
- If the plugin is **NOT installed**, output:
  > ❌ **Error:** The 'Open Knowledge Format' plugin is not installed yet. Install it with `/plugin marketplace add scaccogatto/okf-skills`, then `/plugin install okf@scaccogatto`, then `/reload-plugins` (see README section 2), and try again.
- Stop execution immediately if the plugin is missing.

### Step 2: Parse Arguments & Validate Inputs
Split the raw arguments on whitespace, in order. A token wrapped in double or single quotes may contain spaces (for example `"my notes.md"`) and counts as one token; strip the quotes:
- `<directory>` = first token (default `docs` if absent)
- `<file>` = second token (default `all` if absent)

If there are more than two tokens, output an error that lists the extra tokens and the usage line `/okf-unverify [directory] [file|all]`, and stop. (This command takes no name: it removes sign-offs, it does not add one.)

`<directory>` and `<file>` are relative to the project root. If `<directory>` does not exist, or `<file>` (when not `all`) does not exist inside it, output an error naming the missing path and stop. A specific `<file>` must be a `.md` file; it may include a sub-path such as `sub/guide.md`.

### Step 3: Interactive Confirmation (If file is 'all')
If `<file>` is `all`:
- Do NOT execute directly.
- Ask the user:
  > ⚠️ By default, this command will **remove the verification** from **all files** within the `<directory>/` directory and set them back to `draft`. Are you sure you want to proceed? (yes/no)
- Then end your turn and wait for the reply. Continue only if the reply is `yes` or `y` (any case). Anything else, or no reply, means exit cleanly without modifying any files.
- If `<file>` is explicitly specified, skip this confirmation and proceed with only that file.

### Step 4: Execution Workflow
1. Build the file list. `all` means every `.md` file in `<directory>`, including subdirectories, except the reserved files `index.md` and `log.md` (§8, §9). Read each file's frontmatter and **skip, and list at the end with the reason**, any file that:
   - is empty, has no frontmatter, or has unparseable or unclosed frontmatter (nothing to remove; tell the user to fix it by hand or run `/okf-docs`);
   - has no `verified` key: it is not verified, so there is nothing to remove;
   - has `status: deprecated`: setting it to `draft` would silently revive a retired document.
2. Before changing a file, note who had signed it off (every `by` in its `verified` entries), so you can list them in the summary.
3. Update each remaining file. Write each file only once. Rules:
   - Delete the whole `verified` key: the `verified:` line and every entry under it (§5.3: no `verified` key means **unverified**). A one-line `verified: { by: ..., at: ... }` mapping is deleted the same way.
   - Set `status: draft` (§5.4: "not yet reviewed"). If `status` is missing (which §5.4 reads as `stable`), add `status: draft` after `tags` (or after the last key if there is no `tags`).
   - Change nothing else: not the body, not `generated` (its `at` changes only when the content changes; the author updates it when editing, or runs `/okf-docs`), not `type`, `title`, `description`, `tags`, `owner` or any other key, and do not rename files.
   - Keep the file's existing line endings (CRLF or LF) and UTF-8.

4. Validate with the plugin's **`okf:validate`** skill: arguments `<directory>/ --strict` for `all`, or `<directory>/ --json` for a single file (the checker only accepts a folder, so judge only the findings for the target file).
   - If the skill's `uv` and `python3` commands fail (common on Windows), run the same script with `python`, then `py -3`. If PyYAML is missing, point the user to README section 2 instead of installing it.
   - `/okf-unverify` only changes `status` and `verified`: report any other finding and tell the user to run `/okf-docs` or fix it by hand.
5. Finish with a short summary: files set back to draft (and whose sign-offs were removed from each), files skipped (and why), the validator result, and the next step: edit the doc, then ask a reviewer to run `/okf-verify <reviewer> <directory> <file>`.
