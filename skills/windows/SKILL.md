---
name: windows
description: Windows-specific problems in development tools. Use when a tool reads or writes text on Windows, when mise shims fail, when long paths, PowerShell quoting or a persistent terminal misbehave, or when a toolchain's PATH order matters.
---

# Tools on Windows

## Text encoding and newlines

- A tool writes text with an explicit encoding and newline, for example `path.write_text(text, encoding="utf-8", newline="\n")`, never the platform's. Without it Windows writes `\r\n` for a `\n` inside a generated multi-line string, and the file is corrupt.
- A tool reads text with an explicit encoding too, for example `path.read_text(encoding="utf-8")`. Without it, a read on Windows decodes with the console codepage (`cp1252`), not UTF-8, so a tracked file that holds a real, valid UTF-8 character outside that codepage raises `UnicodeDecodeError`, deterministically, on every run on that platform. It was found in a check that read a tracked file with a bare `Path.read_text()` and failed two tests on every Windows checkout for a reason that read as flaky until traced.

## Shims and PATH

- **A shim is not the binary.** `mise`'s shims re-invoke `mise`, so they fail with "program not found" until the real `mise.exe` is on `PATH`. A newly installed `mise` is not on `PATH` in a terminal opened before the install.
- **opam's Cygwin before Git's** on `PATH`, or native OCaml's assembler and linker fail.
- **`openssl`** ships with Git for Windows if it is not otherwise on `PATH`. Add it at user level, never by hard-coding a path into a task.

## Long paths and toolchains

- **Long paths are often off.** Install large toolchains, such as GHC or opam, under a short root.

## PowerShell

- **PowerShell `-File` passes `-X a,b` as one string**, and variable names are case-insensitive.
- **Write multi-line Python to a temporary file** rather than passing it to `python -c` from PowerShell, whose quoting mangles embedded quotes and newlines.
- **A long multi-line PowerShell command block sent to a persistent terminal** can appear to echo stale output from an earlier command, or seem to hang, when it is in fact still executing. Check the actual redirected output file directly (`Get-Content <file> -Tail N`) rather than trusting the terminal capture buffer for a long-running background command.
