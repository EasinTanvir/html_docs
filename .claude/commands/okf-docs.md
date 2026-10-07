---
description: Convert documentation files to OKF v0.2 format as a draft using the Open Knowledge Format plugin.
argument-hint: <author_name> [directory] [file|all]
---

Raw arguments: `$ARGUMENTS`

Perform the following steps carefully. Parse the raw arguments above yourself, as described in Step 2.

### Step 1: Pre-requisite Check
Check if the **Open Knowledge Format** plugin (`okf@scaccogatto`) is installed. The plugin counts as installed if the `okf:okf` skill is available in this session. If you cannot tell, look for `okf@scaccogatto` in `~/.claude/plugins/installed_plugins.json`, but only count an entry whose scope is `user`, or whose `projectPath` is the current project (an entry for another project does not enable the plugin here). A user-scope or project-scope install both work.
- If the plugin is **NOT installed**, output:
  > ❌ **Error:** The 'Open Knowledge Format' plugin is not installed yet. Please install it first using `/plugin marketplace add scaccogatto/okf-skills`, then `/plugin install okf@scaccogatto`, then `/reload-plugins`, and try again.
- Stop execution immediately if the plugin is missing.

### Step 2: Parse Arguments & Validate Mandatory Inputs
Split the raw arguments on whitespace, in order. A token wrapped in double quotes may contain spaces (for example `"my notes.md"`) and counts as one token; strip the quotes:
- `<author>` = first token (**MANDATORY**, no fallback default)
- `<directory>` = second token (default `docs` if absent)
- `<file>` = third token (default `all` if absent)

If there are more than three tokens, output an error that lists the extra tokens and the usage line, and stop.

If `<author>` is missing:
- Output:
  > ❌ **Error:** Author name is required as the first argument. Usage: `/okf-docs <author_name> [directory] [file|all]`
- Stop execution immediately.

`<author>` must be a single token of letters, digits, `.`, `_` or `-` (it becomes the `human:<author>` actor). If it contains anything else, output an error saying so and stop.

If `<directory>` does not exist, or `<file>` (when not `all`) does not exist inside it, output an error naming the missing path and stop. A specific `<file>` must be a `.md` file.

### Step 3: Interactive Confirmation (If file is 'all')
If `<file>` is `all`:
- Do NOT execute directly.
- Ask the user:
  > ⚠️ By default, this command will process **all files** within the `<directory>/` directory for author `human:<author>`. Are you sure you want to proceed? (yes/no)
- If the user responds with anything other than 'yes', exit cleanly without modifying any files.
- If `<file>` is explicitly specified (e.g., `/okf-docs EasinTanvir docs email-setup.md`), skip this confirmation and proceed directly with only that file.

### Step 4: Execution Workflow
1. Build the file list. `all` means every `.md` file in `<directory>`, including subdirectories. Never convert the reserved files `index.md` and `log.md` (they must have no frontmatter) and ignore non-Markdown files. **Skip, and list at the end with the reason**, any file that:
   - is already verified (it has a `verified` block): converting it again would reset a human sign-off. Tell the user to edit such a file by hand if it really needs to go back to draft;
   - has `status: deprecated`: converting it would silently revive a retired document;
   - is empty or contains only whitespace: there is nothing to convert;
   - starts with a `---` line but has no closing `---`, or has frontmatter that is not valid YAML: do not guess, tell the user to fix the frontmatter by hand.
2. Invoke the installed **Open Knowledge Format (`okf:okf`)** plugin skill (maintain mode) to process each file in the list.
3. Rules for the conversion:
   - Add OKF v0.2 YAML frontmatter with the required `type` field, set to the Diátaxis type that fits the content (`Runbook`, `Guide`, `Reference`, or `Explanation`).
   - Set `title` (the file's top heading, or a readable name from the file name) and `description` (one sentence summarizing the document). Both are recommended by the spec and the strict validation below fails without them. Always write both as double-quoted YAML strings (escape inner double quotes) so characters such as `:`, `#` and `&` stay valid.
   - If the file has legacy v0.1 constructs (a `timestamp` field or a body `# Citations` list), first copy that one file into a temporary directory, run the `okf:validate` skill on that temporary directory with the argument `--migrate`, and copy the file back. On Windows `--migrate` rewrites the file with CRLF line endings, so convert it back to the file's original line endings afterwards. Then continue with the rules below (`generated` is set to the human author).
   - Keep the file's existing line endings (CRLF or LF) for every line you add, so the file does not end up with mixed endings.
   - If the file already has frontmatter, keep its existing fields (`resource`, `sources`, extra tags and so on) and only add or correct the fields in this list. Do not duplicate keys.
   - Set `owner: human:<author>`.
   - Set `status: draft`.
   - Set `generated: { by: "human:<author>", at: "<CURRENT_DATE>" }`, where the date is the current system date as `YYYY-MM-DD`.
   - Add relevant `tags`.
   - Omit the `verified` block completely, so it stays open for peer review.
   - Restructure the body into its Diátaxis type only where needed, without altering the factual content or changing file names.

4. Run the OKF validation with the plugin's own **`okf:validate`** skill (do not hard-code any path to the checker; the skill finds it by itself and handles `uv` or Python and PyYAML):
   - If `<file>` is `all`: invoke `okf:validate` with the arguments `<directory>/ --strict`.
   - If `<file>` is specific: the checker only accepts a directory, so invoke `okf:validate` with `<directory>/ --json` and consider only the `errors` and `warnings` entries that start with the target file's path inside the directory (for example `runbook.md:` or `sub/guide.md:`). Findings for other files are not yours to fix; mention them in one line. Pass only if there are no entries for the target file.
   - The skill prints a `uv run ...` command and a `python3 ...` fallback. If `uv` is not installed and `python3` fails (on Windows `python3` is often a Microsoft Store stub that prints "Python was not found"), run the same script path with `python` instead, after `python -m pip install --quiet pyyaml`. This is normal, not an error.
   - Only if the `okf:validate` skill is not available: find `okf_validate.py` anywhere under `~/.claude/plugins/` (use the highest version if there are several), run it with `python`, and if that fails with `No module named 'yaml'`, run `python -m pip install --user pyyaml` and try again. Never use `python -I`.
   - `--strict` (for `all`) makes warnings fail the run, so fix every error and warning before finishing. Broken cross-links to files outside the target are the only thing you may leave, and you must report them.
   - Files you skipped because they are empty or have broken frontmatter will still make the validator report errors. That is expected: do not try to fix them, list them under the skipped files with the fix the user needs to make, and say that the remaining validator findings are only for those files.
   - Finish with a short summary: files converted, files skipped (and why), and the validator result.
