---
name: mise
description: Using mise to run a project's build, test and check tasks, setting up shells and keeping the code under test the checkout's own. Use when running or choosing mise tasks, when a build, test or package install problem appears, when adding a mise task, or before running anything that imports the project's Python packages.
---

# The mise toolchain

This skill's tasks span the activities environment, build-automation, dependency-management and continuous-integration, and a task often needs several of them, named together and separated by commas.

The [developer guide]({{developer_guide}}) describes every tool and task, and [getting started]({{getting_started}}) the first setup. Read them rather than guessing a command. Lessons from past environment problems are in the skill `environment-lessons`.

## Running tasks

- `mise tasks` lists a project's tasks, and `mise run <task>` runs one with the tools and environment `mise.toml` declares.
- A task that reads an artefact a build writes names that build in its `depends`. A suite failing on a missing generated file is an undeclared dependency to add in `mise.toml`, never a file to stage by hand.
- `mise run` stops every sibling task as soon as one fails, so a known-broken task hides the rest. `mise run --continue-on-error <task>` runs them all and reports every failure (NOTE-11).

## Is the code under test this checkout's?

Where `mise.toml` creates a virtual environment per checkout (`_.python.venv`), `mise run`, `mise exec` and an activated shell use it, and a bare `python` in a shell without mise activated is the global interpreter, whose editable installs can point at another clone. So run Python through `mise run` or `mise exec -- python`, and before running anything that imports {{packages}}:

```bash
mise exec -- python -c "{{import_check}}"
```

Every path must be inside this checkout. If not, or if the import fails, install the project's packages again in this checkout.

## Shell activation

Installing `mise` is not enough. A new shell has no `mise`-managed tools on `PATH` until its profile activates it.

- POSIX shells: `eval "$(mise activate zsh)"` (or `bash`) in the rc file.
- PowerShell: `mise activate pwsh | Out-String | Invoke-Expression` in `$PROFILE`. On Windows PowerShell 5.1, set `$env:MISE_PWSH_CHPWD_WARNING = "0"` first.
- If a package manager put `mise` somewhere not on `PATH`, find the binary and add its directory first. Verify in a brand-new shell, never by patching the current one.

## Rules for tasks and tools

- `mise.toml` `run` strings run under `cmd.exe` on Windows and a POSIX shell elsewhere. Use double quotes around arguments with shell-special characters (`"./pkg[test]"`), and never rely on a shell glob. Enumerate files explicitly.
- A task that needs more than plain commands, such as a variable, a loop, a redirect or a multi-line script, or that reads or writes text whose encoding or line endings matter, is portable as a script in a language the project already runs, not as inline shell. A sequence of plain commands is a `run` array, one command per entry, with `dir` instead of `cd`.
- Where a task truly has no portable form, such as a platform's own installer, its Windows variant is `run_windows` with `shell = "powershell -c"`. `$ErrorActionPreference = "SilentlyContinue"` with a closing `exit 0` suits only a clean-up that may find nothing to remove, since it reports success whatever failed.
- A tool that prints a repository-relative path for comparison prints `Path.as_posix()`.
- After changing a list other code counts or iterates over, such as a task's test modules, a version pin, or a set whose size a test asserts, run every task and test that reads that list, not only the one you edited. A merge resolution once left a task skipping two test modules without failing, and a contract added beyond a pinned count was caught only by the full sweep.
