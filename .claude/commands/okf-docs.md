---
description: Convert documentation files to OKF v0.2 format as a draft using the Open Knowledge Format plugin.
argument-hint: <author_name> [directory] [file|all]
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
- `<author>` = first token (**MANDATORY**, no fallback default)
- `<directory>` = second token (default `docs` if absent)
- `<file>` = third token (default `all` if absent)

If there are more than three tokens, output an error that lists the extra tokens and the usage line, and stop.

If `<author>` is missing:
- Output:
  > ❌ **Error:** Author name is required as the first argument. Usage: `/okf-docs <author_name> [directory] [file|all]`
- Stop execution immediately.

`<author>` must be a single token of letters, digits, `.`, `_` or `-` (it becomes the `human:<author>` actor, §7). If it contains anything else, output an error saying so and stop. Use it exactly as typed (do not change its case).

`<directory>` and `<file>` are relative to the project root. If `<directory>` does not exist, or `<file>` (when not `all`) does not exist inside it, output an error naming the missing path and stop. A specific `<file>` must be a `.md` file; it may include a sub-path such as `sub/guide.md`.

### Step 3: Interactive Confirmation (If file is 'all')
If `<file>` is `all`:
- Do NOT execute directly.
- Ask the user:
  > ⚠️ By default, this command will process **all files** within the `<directory>/` directory for author `human:<author>`. Are you sure you want to proceed? (yes/no)
- Then end your turn and wait for the reply. Continue only if the reply is `yes` or `y` (any case). Anything else, or no reply, means exit cleanly without modifying any files.
- If `<file>` is explicitly specified (e.g., `/okf-docs EasinTanvir docs email-setup.md`), skip this confirmation and proceed directly with only that file.

### Step 4: Execution Workflow
1. Build the file list. `all` means every `.md` file in `<directory>`, including subdirectories. Never convert the reserved files `index.md` and `log.md` (§8, §9: they are not concepts and `log.md` has no frontmatter) and ignore non-Markdown files. **Skip, and list at the end with the reason**, any file that:
   - is already verified (its frontmatter has a `verified` key): converting it again would reset a human sign-off. Tell the user to edit such a file by hand if it really needs to go back to draft;
   - has `status: deprecated`: converting it would silently revive a retired document;
   - is empty or contains only whitespace: there is nothing to convert;
   - starts with a `---` line but has no closing `---` line, or has frontmatter that is not valid YAML or not a mapping: do not guess, tell the user to fix the frontmatter by hand.
2. Get the current time **once**, in UTC, as an RFC 3339 timestamp (§5: "Every timestamp-valued key in OKF is an ISO 8601 datetime with an explicit UTC offset"), for example `2026-10-07T09:15:00Z`. Run the first of these that works and use its output as `<NOW>` for every file in this run:
   - `date -u +%Y-%m-%dT%H:%M:%SZ` (macOS, Linux, Git Bash)
   - `powershell -NoProfile -Command "[DateTime]::UtcNow.ToString('yyyy-MM-ddTHH:mm:ssZ')"` (Windows)
   Never invent or guess the time.
3. Use the installed **Open Knowledge Format (`okf:okf`)** skill as the reference for OKF rules, and convert each file in the list by hand with the rules below. Write each file only once, when its new content is complete.
4. Conversion rules:
   - **The body stays exactly as it is.** Do not reword, reformat, add headings, reorder sections or fix typos, and do not rename files. The only body change ever allowed is the v0.1 `# Citations` migration below.
   - Write the frontmatter at the very top of the file, with exactly these keys in exactly this order and this style (the style of the spec's own examples, §5.2):
     ```yaml
     ---
     type: <Type>
     title: "<Title>"
     description: "<One sentence.>"
     tags: [<tag>, <tag>, <tag>]
     status: draft
     generated: { by: human:<author>, at: <NOW> }
     ---
     ```
     followed by one empty line and then the original body (as in the spec's examples, §4.3; do not add the empty line if the body already starts with one). Do not add `verified` (the file stays open for peer review), `owner`, `stale_after`, or any other key that is not in this template.
   - `type` (the only required key, §4.1): the Diátaxis type that fits the content, exactly one of `Runbook`, `Guide`, `Reference`, `Explanation`. Type values are free-form in the spec (§4.1); the team uses these four.
   - `title`: the text of the file's first `#` heading. If there is none, a readable name from the file name (`email-setup.md` gives `Email Setup`).
   - `description`: one sentence that summarizes the document, ending with a period.
   - `title` and `description` are always written as double-quoted YAML strings, so characters such as `:`, `#`, `&` and `'` stay valid. Escape an inner `"` as `\"` and a `\` as `\\`.
   - `tags`: 2 to 5 lowercase tags, words joined with `-` (for example `contact-form`), in a one-line YAML list.
   - `<author>` is written exactly as given in Step 2, after the lowercase `human:` prefix. Do not quote the actor or the timestamp, as in the spec's examples.
   - **If the file already has frontmatter:** keep every existing key and its value (for example `resource`, `sources`, `owner`, custom keys) in its existing place, then make sure the template keys are present: add missing ones and correct invalid ones (`status` not one of `draft | stable | deprecated` becomes `draft`; an existing `stable` also becomes `draft`, and say so in the summary). If the file already has a valid `generated` block, keep it unchanged. Keep existing `title`, `description` and `tags` if they are valid. Never create a duplicate key.
   - **Legacy v0.1 files** (§13.1). Apply the same mapping as the plugin's `--migrate`, by hand:
     - A `timestamp: <value>` key is replaced by `generated: { by: human:<author>, at: <value> }`. Keep the original value as `at`, because it records when the content last changed; if it is a bare date such as `2025-01-01`, write `2025-01-01T00:00:00Z`. If the file already has `generated`, just delete `timestamp`.
     - A body section headed `# Citations` (any heading level) whose list contains Markdown links is moved into a `sources` key placed after `generated`, one entry per link: `  - { resource: "<url>", title: "<link text>" }`. Remove that section from the body (up to the next heading or the end of the file). If the section contains no Markdown links, leave it in place and report it.
     - If `okf_version: 0.1` appears, the file is a bundle-root `index.md`: it is reserved and not converted.
   - **Line endings and encoding:** if the file uses CRLF line endings, every line you add uses CRLF too; otherwise LF. Never mix them. Keep UTF-8. If the file starts with a UTF-8 byte-order mark, remove the mark (the validator cannot see frontmatter behind it).

5. Validate with the plugin's **`okf:validate`** skill: arguments `<directory>/ --strict` for `all`, or `<directory>/ --json` for a single file (the checker only accepts a folder, so judge only the findings for the target file).
   - If the skill's `uv` and `python3` commands fail (common on Windows), run the same script with `python`, then `py -3`. If PyYAML is missing, point the user to README section 2 instead of installing it.
   - Fix every finding your conversion caused, without touching body text. Findings for skipped files or other files are expected: report them, do not edit those files.
6. Finish with a short summary: files converted (with the `type` chosen), files skipped (and why), and the validator result.
