# Prompt for the reviewing agent

Copy everything below the line into the other agent.

---

You are an independent reviewer. Do not trust the author's claims: check everything yourself against the official sources and by running things. Report what is wrong, not what is right.

## 1. What you are reviewing

A project at `D:\docs\New folder` contains two custom Claude Code slash commands and a README that a team will share. The commands convert normal Markdown docs into **OKF v0.2** (Open Knowledge Format) files and let a reviewer sign them off.

| File | Purpose |
|---|---|
| `.claude/commands/okf-docs.md` | `/okf-docs <author> [directory] [file\|all]`: adds OKF frontmatter and sets `status: draft` |
| `.claude/commands/okf-verify.md` | `/okf-verify <verifier> [directory] [file\|all]`: sets `status: stable` and records a `verified` sign-off |
| `README.md` | Step-by-step guide for junior teammates (prerequisites, usage, errors) |
| `docs/overview.md`, `docs/customization.md` | The real sample docs the commands were run on |
| `form.html` | The HTML form the docs describe (context only) |

Goal: the team must be able to share these two commands and get **standard, spec-conformant OKF output every time**, on a machine that is not the author's.

## 2. Official sources (read these first, in this order)

1. OKF v0.2 specification, verbatim, in the installed plugin: `C:\Users\tiger\.claude\plugins\cache\scaccogatto\okf\0.10.0\skills\okf\reference\SPEC.md`
2. Plugin repo: https://github.com/scaccogatto/okf-skills (install steps, skills, validator, actor convention)
3. Plugin skills (installed copies):
   - `C:\Users\tiger\.claude\plugins\cache\scaccogatto\okf\0.10.0\skills\okf\SKILL.md`
   - `C:\Users\tiger\.claude\plugins\cache\scaccogatto\okf\0.10.0\skills\validate\SKILL.md`
   - `C:\Users\tiger\.claude\plugins\cache\scaccogatto\okf\0.10.0\skills\validate\scripts\okf_validate.py` (the validator)
4. Google's OKF repository: https://github.com/GoogleCloudPlatform/open-knowledge-format
5. Google blog post: https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing
6. Claude Code documentation on custom slash commands and plugins (use the official docs at docs.claude.com / code.claude.com): how command arguments work (`$ARGUMENTS`, `$1`, `$N`), where command files live, and how plugin scopes (user vs project) work.

If a page cannot be fetched, say so and fall back to the local SPEC.md. Do not guess spec content.

## 3. Your tasks

### A. Spec conformance of the commands
Read both command files fully. Check every rule they impose against the spec and report any rule that is wrong, missing, or contradicts the spec. Specifically check:
- Required and recommended frontmatter fields (`type`, `title`, `description`, `tags`, `status`, `generated`, `verified`, `stale_after`).
- Allowed `status` values (the author believes only `draft | stable | deprecated`; `active` is invalid).
- The actor convention `human:<id>` (exact lowercase prefix, section 7 of the spec) and the `generated: { by, at }` shape.
- Whether `verified` should be a mapping or a list, and whether appending entries is the right behaviour.
- Reserved files `index.md` and `log.md` (must not be converted, and have no frontmatter).
- Whether `owner` is a legitimate field (the author believes it is non-standard but tolerated). Confirm in the spec which unknown keys are allowed.
- Whether using Diataxis names (`Runbook`, `Guide`, `Reference`, `Explanation`) as the `type` value is allowed and sensible under the spec.
- Legacy v0.1 migration (`timestamp`, `# Citations`) and the `--migrate` behaviour.

### B. Command design and robustness
- Argument parsing: the commands parse `$ARGUMENTS` themselves instead of `$1/$2/$3`. Verify against the Claude Code docs why that is (or is not) necessary, and whether quoting (`"spaced name.md"`) is handled consistently.
- Confirmation flow for `all`, the peer-review (owner equals verifier) check, and the skip rules (already verified, deprecated, empty, broken frontmatter). Look for loopholes, ambiguity, or steps a model could reasonably misread.
- The validation step: it now calls the plugin's own `okf:validate` skill and falls back to `python`. Check that the instructions make sense on a machine with only Python installed (no `uv`) and note anything that would break.
- Anything that could damage a teammate's files: overwriting sign-offs, mixed line endings, changed body text, renamed files, partial writes when one file fails.

### C. README accuracy
- Every install command, version requirement (the author claims Python 3.11 or newer) and check command in section 2 of `README.md` must be verified against the plugin repo and the validator header.
- Every statement about what the commands do must match the command files. List any mismatch.
- Check that a junior teammate with a clean machine could follow it, and name the first step where they would get stuck.

### D. Functional tests (in a scratch copy only)
Do **not** modify anything in `D:\docs\New folder`. Copy the project to a temporary folder and run tests there. Use real slash commands if you can, and say clearly when you ran a step by hand instead. Minimum cases:

1. No arguments; invalid name (`bad:name`); too many arguments; missing directory; missing file; a file that is not `.md`.
2. Convert one file, then all files; confirm the prompt appears for `all` and not for a single file; answer "no" and confirm nothing changes.
3. Files with existing frontmatter, CRLF endings, a colon in the title (`Setup: Mail & Queue`), Unicode, no heading, nested folders, an empty file, a file with unclosed frontmatter, a `deprecated` file, a legacy v0.1 file.
4. Verify as a different person; then verify again as a third person (must append, not overwrite); then verify as the owner (must warn and ask).
5. Run the validator with `--strict` on every result and report the exact output.
6. Re-run `/okf-docs` on a verified file (must skip it).

For each case record: command, expected result, actual result, pass or fail.

## 4. Known facts and claims to confirm or refute

The author found and fixed these. Confirm each one independently, and say if any is wrong:

1. In the author's Claude Code version, `$1` held the second argument and `$2`/`$3` stayed as literal text, so the commands parse `$ARGUMENTS` themselves.
2. `status: active` is not valid OKF. The valid values are `draft`, `stable`, `deprecated`.
3. The validator accepts only a directory, never a single file, so single-file checks validate the directory and filter the findings by file path (`--json`).
4. On Windows, `--migrate` rewrites an LF file with CRLF line endings.
5. On Windows, `python3` can be a Microsoft Store stub, and `uv` may be missing. `python` plus `pip install pyyaml` works.
6. `title` and `description` are only "recommended", but `--strict` turns their absence into a failure, so the commands always add them.
7. The `okf:okf` skill and `okf@scaccogatto` entries in `~/.claude/plugins/installed_plugins.json` exist for both user-scope and project-scope installs, and both point to the same cache folder.

## 5. Things the author could not verify (please test if you can)

- Mac and Linux behaviour (the author tested Windows 11 only).
- A truly clean machine: no plugin, no PyYAML, no `uv`.
- A project-scope install by a second person.
- Whether the confirmation prompts behave the same in a non-interactive or headless run.

## 6. Known open problem (not caused by the commands)

The files in `docs/` have repeatedly lost their frontmatter outside the author's session, and `docs/customization.md` is missing its first ~12 lines (title and the first two sections). `docs/` is untracked in git. Do not "fix" it; just note whether anything in the commands could plausibly cause this. (The commands only add frontmatter and never edit body text; confirm that claim.)

## 7. Output format

Write your report in this structure:

1. **Verdict:** safe to share / safe after fixes / not safe, in one line.
2. **Blocking issues:** each with file, line, what is wrong, the official source that proves it (link or spec section), and a proposed fix.
3. **Non-blocking issues and suggestions.**
4. **Claims confirmed or refuted** (section 4, one line each).
5. **Test results table** (section 3D).
6. **Not verified:** what you could not check and why.

Rules: cite the spec section or URL for every claim about OKF. Quote the exact line of the command or README you are criticising. Do not edit the real files. Do not invent test results: if you did not run it, say so.
