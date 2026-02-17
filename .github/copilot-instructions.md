# This project-specific
This is a multi-root project with roots at
  * `github.com\LogicDaemon\ScoopInstaller-manifest-fixes`
  * `github.com\ScoopInstaller\Scoop`
  * `Programs\scoop\buckets\main`
  * `Programs\scoop\buckets\extras`
check the specific workspace contents when starting working on it to find out the exact paths in the filesystem.

- You're only allowed to make changes in ScoopInstaller-manifest-fixes. Use other repositories as reference and examples on implementation.
- For Scoop manifest edits, apply the dedicated file instruction at `.github/instructions/scoop-manifest-authoring.instructions.md`.

# General GitHub Copilot Instructions
- Avoid AI sycophancy. Even if there could be multiple views or the user gets easily offended, at least express a mild doubt
- Avoid `run_in_terminal` when more targeted tool is available:
  * `read_file` not `cat`
  * `list_dir` not `ls`
  * `file_search`, `grep_search`, `semantic_search` instead of `grep`
  * `replace_string_in_file` not `sed`
  * etc: `create_directory`, `create_file`, `fetch_webpage`, `get_changed_files`, `get_errors`, `list_code_usages`
- When `run_in_terminal` is necessary, prefer Python over shell commands
- Before committing, run `git status` and `git diff` to ensure only the intended changes are included
- Check the current directory (`pwd`) or execute `cd` before the shell commands, it's a multi-repository environment, do not assume the default directory is what you expect

## Coding Conventions
- Do not remove comments you did not add
- Avoid Speculative Safety: evaluate code by context, not hypothetical reuse (e.g. python aliasing is acceptable if it works)
- No Premature Generalization
- When updating the repository, always update this documentation to reflect any changes in architecture or conventions
  * But keep this file concise. Document nuances that are not obvious from scanning the code, but skip trivia or restating implementation details visible in the modules

## Complex tasks approach
- Create a `.github/task_<name>.md` file with:
  1. Task description — it should be clear what the end goal is. It's written once and not updated
  2. Current state analysis — what is the current situation, what are the constraints, what has been tried already (if applicable). This should be updated as you learn more, and must be updated after each failed attempt
  3. Step-by-step plan, each step with a verifiable outcome. After implementing each step, mark it as "verifying" and test. If it works, mark it as "done" and proceed to the next step. If it doesn't work, update the current state analysis, adjust the plan accordingly and undo any changes from the failed step
- When starting/continuing a task, review the file and re-analyze the repository contents to understand the context and avoid repeating past mistakes, and to check if the file is up-to-date
- When the task is done, append the summary to the file
- This file should be stored alongside the code changes until the task is done. Do not remove it yourself, let the user decide when to delete it
