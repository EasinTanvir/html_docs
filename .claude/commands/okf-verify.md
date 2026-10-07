---
description: Verify an OKF document and mark it stable (peer-reviewed) using the Open Knowledge Format plugin.
argument-hint: <verifier_name> [directory] [file|all]
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
- `<verifier>` = first token (**MANDATORY**, no fallback default)
- `<directory>` = second token (default `docs` if absent)
- `<file>` = third token (default `all` if absent)

If there are more than three tokens, output an error that lists the extra tokens and the usage line, and stop.

If `<verifier>` is missing:
- Output:
  > ❌ **Error:** Verifier name is required as the first argument. Usage: `/okf-verify <verifier_name> [directory] [file|all]`
- Stop execution immediately.

`<verifier>` must be a single token of letters, digits, `.`, `_` or `-` (it becomes the `human:<verifier>` actor). If it contains anything else, output an error saying so and stop.

If `<directory>` does not exist, or `<file>` (when not `all`) does not exist inside it, output an error naming the missing path and stop. A specific `<file>` must be a `.md` file.

### Step 3: Interactive Confirmation (If file is 'all')
If `<file>` is `all`:
- Do NOT execute directly.
- Ask the user:
  > ⚠️ By default, this command will verify **all files** within the `<directory>/` directory for verifier `human:<verifier>`. Are you sure you want to proceed? (yes/no)
- If the user responds with anything other than 'yes', exit cleanly without modifying any files.
- If `<file>` is explicitly specified, skip this confirmation.

### Step 4: Execution Workflow
1. Build the file list. `all` means every `.md` file in `<directory>`, including subdirectories, except the reserved files `index.md` and `log.md`. Read each file's frontmatter and **skip, and list at the end with the reason**, any file that:
   - has no OKF frontmatter, has frontmatter without a `type`, or has unparseable or unclosed frontmatter (tell the user to run `/okf-docs` on it, or fix it by hand);
   - is empty;
   - has `status: deprecated`;
   - is already `status: stable` and already verified by `human:<verifier>`.
2. Peer-review check: if a file's `owner` is `human:<verifier>`, warn that the verifier is the owner (verification is meant to be peer review) and ask the user to confirm for that file before changing it. If they decline, skip it.
3. Invoke the installed **Open Knowledge Format (`okf:okf`)** plugin skill (maintain mode) to update each remaining file. Rules:
   - Set `status: stable`. This is the spec's value for "ready for consumption"; `active` is NOT a valid OKF status.
   - Record the sign-off in `verified` as a list entry `{ by: "human:<verifier>", at: "<CURRENT_DATE>" }`, where the date is the current system date as `YYYY-MM-DD`. If `verified` is absent, create it. If it already holds earlier entries (a single mapping or a list), keep them and append the new entry; never overwrite another person's sign-off.
   - Do not change the body, `generated`, `owner`, or any other field, and do not rename files. Keep the file's existing line endings (CRLF or LF).

4. Run the OKF validation script to verify compliance:
   - Locate the script at `~/.claude/plugins/cache/scaccogatto/okf/*/skills/validate/scripts/okf_validate.py` (use the highest version). If it is not there, look under `~/.claude/plugins/marketplaces/scaccogatto/skills/validate/scripts/`.
   - If `<file>` is `all`: `python <script> <directory>/ --strict`
   - If `<file>` is specific: the validator only accepts a directory, so run `python <script> <directory>/ --json` and consider only the `errors` and `warnings` entries that start with the target file's path inside the directory (for example `runbook.md:` or `sub/guide.md:`). Findings for other files are not yours to fix; mention them in one line. Pass only if there are no entries for the target file.
   - Run it as plain `python <script> ...`, not `python -I`, because the script needs PyYAML from the user's site-packages. If it fails with `No module named 'yaml'`, run `python -m pip install --user pyyaml` and try again.
   - `--strict` (for `all`) makes warnings fail the run, so fix every error and warning before finishing. Broken cross-links to files outside the target are the only thing you may leave, and you must report them.
   - Finish with a short summary: files verified, files skipped (and why), and the validator result.
