---
name: environment-lessons
description: Lessons from problems met with Python scripts, uv environments, Node tooling and JVM libraries while developing. Use when a Python script or test must run where packages cannot install, when a uv environment is corrupt, when yarn or Playwright misbehave, or when a JVM library's behaviour is unclear.
---

# Environment lessons

Problems met while developing, kept for the next agent who meets them. Each is general. A note true of one machine belongs in that person's own configuration, not here.

## Python

- **Stdlib-only tools still run where packages cannot install.** A script that needs only the standard library runs when nothing can be installed. Check before concluding that nothing can be run.
- **An AST-scan test can run without pytest**, by importing the test module and calling the test function, which is useful for probing a test against a deliberately broken implementation.

## Node and the frontends

- **A `.venv` made by `uv` has no pip, and parallel pip corrupts one.** Every Python bootstrap task depends on one that installs pip into it. Run several of them with `mise run --jobs 1`, because concurrent installs of two versions of a package (`pytest` showed it, as an `ImportError` in `_pytest`) interleave files. The repair is `rm -rf .venv && mise run bootstrap`.
- **`yarn --version` can block** at Corepack's download prompt and swallow the next command as its answer. Set `COREPACK_ENABLE_DOWNLOAD_PROMPT=0` first.
- **Playwright clears its output directory**, deleting the `.gitkeep` in `{{test_results}}`. Restore it with `git checkout` before committing.

## JVM libraries

- **networknt's JSON Schema validator** can preload a set of schemas by `$id`, with no network fetch, through `JsonSchemaFactory.builder(…).schemaLoaders(l -> l.schemas(map))`.
