---
description: Verify an OKF document and mark it stable (peer-reviewed) using the Open Knowledge Format plugin.
argument-hint: <verifier_name> [directory] [file|all]
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

### Step 2: Parse Arguments & Validate Mandatory Inputs
Split the raw arguments on whitespace, in order. A token wrapped in double or single quotes may contain spaces (for example `"my notes.md"`) and counts as one token; strip the quotes:
- `<verifier>` = first token (**MANDATORY**, no fallback default)
- `<directory>` = second token (default `docs` if absent)
- `<file>` = third token (default `all` if absent)

If there are more than three tokens, output an error that lists the extra tokens and the usage line, and stop.

If `<verifier>` is missing:
- Output:
  > ❌ **Error:** Verifier name is required as the first argument. Usage: `/okf-verify <verifier_name> [directory] [file|all]`
- Stop execution immediately.

`<verifier>` must be a single token of letters, digits, `.`, `_` or `-` (it becomes the `human:<verifier>` actor, §7). If it contains anything else, output an error saying so and stop. Use it exactly as typed (do not change its case).

`<directory>` and `<file>` are relative to the project root. If `<directory>` does not exist, or `<file>` (when not `all`) does not exist inside it, output an error naming the missing path and stop. A specific `<file>` must be a `.md` file; it may include a sub-path such as `sub/guide.md`.

### Step 3: Interactive Confirmation (If file is 'all')
If `<file>` is `all`:
- Do NOT execute directly.
- Ask the user:
  > ⚠️ By default, this command will verify **all files** within the `<directory>/` directory for verifier `human:<verifier>`. Are you sure you want to proceed? (yes/no)
- Then end your turn and wait for the reply. Continue only if the reply is `yes` or `y` (any case). Anything else, or no reply, means exit cleanly without modifying any files.
- If `<file>` is explicitly specified, skip this confirmation.

### Step 4: Execution Workflow
1. Build the file list. `all` means every `.md` file in `<directory>`, including subdirectories, except the reserved files `index.md` and `log.md` (§8, §9). Read each file's frontmatter and **skip, and list at the end with the reason**, any file that:
   - is empty, has no frontmatter, has frontmatter without a non-empty `type`, or has unparseable or unclosed frontmatter (tell the user to run `/okf-docs` on it first, or fix it by hand);
   - has `status: deprecated`;
   - already has a `verified` entry whose `by` is `human:<verifier>` and has `status: stable` (or no `status`, which means `stable`, §5.4): it is already signed off by this person.
2. Peer-review check: the author of a file is the actor in `generated.by` (and `owner`, if the file has that key). If either one is `human:<verifier>` (compare case-insensitively), warn that the verifier is the author (verification is meant to be peer review), ask the user to confirm for that file with yes/no, and end your turn to wait for the reply. Continue for that file only on `yes` or `y`; otherwise skip it and list it. When several files need this, ask about all of them in one message.
3. Get the current time **once**, in UTC, as an RFC 3339 timestamp (§5), for example `2026-10-07T09:15:00Z`. Run the first of these that works and use its output as `<NOW>`:
   - `date -u +%Y-%m-%dT%H:%M:%SZ` (macOS, Linux, Git Bash)
   - `powershell -NoProfile -Command "[DateTime]::UtcNow.ToString('yyyy-MM-ddTHH:mm:ssZ')"` (Windows)
   Never invent or guess the time.
4. Update each remaining file by hand (use the installed **`okf:okf`** skill as the reference for OKF rules). Write each file only once. Rules:
   - Set `status: stable` (§5.4: `draft | stable | deprecated`; `stable` means "ready for consumption"). Any other value, including `active`, is not valid OKF. If `status` is missing, add it after `tags` (or after the last key if there is no `tags`).
   - Record the sign-off in `verified` (§5.2), written in the list style of the spec:
     ```yaml
     verified:
       - { by: human:<verifier>, at: <NOW> }
     ```
     If `verified` is absent, add it right after `generated` (or after `status` if there is no `generated`). If it already holds earlier entries (a list, or a single `{ by, at }` mapping, which §5.2 says means a one-element list), keep every earlier entry unchanged, rewrite a single mapping as the first list item, and append the new entry at the end. Never remove or overwrite another person's sign-off.
   - Change nothing else: not the body, not `generated` (§5.2: `verified` is independent of `generated.at`), not `owner`, not any other key, and do not rename files.
   - Keep the file's existing line endings (CRLF or LF) for every line you add or change, and keep UTF-8.

5. Validate with the plugin's **`okf:validate`** skill: arguments `<directory>/ --strict` for `all`, or `<directory>/ --json` for a single file (the checker only accepts a folder, so judge only the findings for the target file).
   - If the skill's `uv` and `python3` commands fail (common on Windows), run the same script with `python`, then `py -3`. If PyYAML is missing, point the user to README section 2 instead of installing it.
   - `/okf-verify` only changes `status` and `verified`: report any other finding and tell the user to run `/okf-docs` or fix it by hand.
6. Finish with a short summary: files verified, files skipped (and why), and the validator result.
