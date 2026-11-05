---
name: mise
description: Using mise as the one entry point for build, test and check tasks, setting up shells and keeping the code under test the checkout's own. Use when running or choosing mise tasks, when a build, test or package install problem appears, when adding a mise task, or before running anything that imports the project's Python packages.
---

# The mise toolchain

This skill's tasks span the activities environment, build-automation, dependency-management and continuous-integration, and a task often needs several of them, named together and separated by commas.

The [developer guide]({{developer_guide}}) describes every tool and task, and [getting started]({{getting_started}}) the first setup. Read them rather than guessing a command. Lessons from past environment problems are in the skill `environment-lessons`.

## mise is the one entry point

- Every build, test and check runs through `mise`. CI runs the same tasks. {{other_authorities}} remain their own dependency authorities.
- `mise install`, `mise run bootstrap`, `mise run check`, and `mise tasks` to list them.
- Those two commands are all `check` needs. A `check:*` task that reads an artefact a build writes names that build in its `depends`. A suite failing on a missing generated file is an undeclared dependency to add in `mise.toml`, never a file to stage by hand.
- Task families are `bootstrap:*`, `check:*`, `build:*` and `clean:*`. Some heavy checks are deliberately outside `check`, as the developer guide lists.
- Run the checks a change needs, not only the aggregate. `mise run check` stops every sibling task as soon as one fails, so a known-broken task hides the rest. Run the remaining `check:*` tasks one by one for a true picture.

## Is the code under test this checkout's?

Each checkout, git worktrees included, has its own `.venv`, which `mise.toml` creates and which `mise run`, `mise exec` and an activated shell use. A bare `python` in a shell without mise activated is the global interpreter, whose editable installs can point at another clone. So run Python through `mise run` or `mise exec -- python`, and before running anything that imports {{packages}}:

```bash
mise exec -- python -c "{{import_check}}"
```

Every path must be inside this checkout. If not, or if the import fails, run `mise run bootstrap` in this checkout.

## A running process and the build

A build never writes a file that a running process has open. A JVM reads classes from its jar for as long as it runs, so a build that rewrites the jar under it leaves it failing with `NoClassDefFoundError` on the first class it had not loaded yet. A process started from the repository runs from a copy of what the build wrote, made just before it starts. A process that has to run from the build output itself is stopped before the build, and the task that starts it says so.

## Shell activation

Installing `mise` is not enough. A new shell has no `mise`-managed tools on `PATH` until its profile activates it.

- POSIX shells: `eval "$(mise activate zsh)"` (or `bash`) in the rc file.
- PowerShell: `mise activate pwsh | Out-String | Invoke-Expression` in `$PROFILE`. On Windows PowerShell 5.1, set `$env:MISE_PWSH_CHPWD_WARNING = "0"` first.
- If a package manager put `mise` somewhere not on `PATH`, find the binary and add its directory first. Verify in a brand-new shell, never by patching the current one.

## Rules for tasks and tools

- `mise.toml` `run` strings run under `cmd.exe` on Windows and a POSIX shell elsewhere. Use double quotes around arguments with shell-special characters (`"./pkg[test]"`), and never rely on a shell glob. Enumerate files explicitly.
- A Windows-only variant of a task uses `run_windows` with `shell = "powershell -c"`. Start its script with `$ErrorActionPreference = "SilentlyContinue"` and end it with `exit 0`, because PowerShell exits 1 after any suppressed non-terminating error.
- A tool that prints a repository-relative path for comparison prints `Path.as_posix()`.
- Ask {{maintainer}} to start Docker Desktop if its engine is not running. Do not start it yourself.
- After changing a list other code counts or iterates over, such as a task's test modules, a version pin, or a set whose size a test asserts, run every task and test that reads that list, not only the one you edited. A merge resolution once left a task skipping two test modules without failing, and a contract added beyond a pinned count was caught only by the full sweep.
