# DevLog Tool

A set of scripts for tracking development progress by wrapping common build/test
commands with git commits and structured logging.

## What it does

Every time you run a wrapped command (like `cargo devlog-test`), the tool will:

1. Prompt you to describe what you've changed
2. Stage all changes and create a commit with your description
3. Run your command (e.g., `cargo run`)
4. Capture the output and git diff
5. Store everything in a TOML file in your project root's `.devlog/*.toml`

## Installation

1. Place the `devlog-wrapper` and `devlog-capture` scripts
somewhere in your PATH (e.g., `~/.local/bin/`)
2. Make them executable: `chmod +x ~/.local/bin/devlog-*`

## Creating Command Wrappers

Create small wrapper scripts for any commands you want to log.
The naming convention is `<tool>-devlog-<command>`:

### For Rust projects

```bash
# ~/.local/bin/cargo-devlog-run
#!/bin/bash
shift
exec devlog-wrapper cargo run "$@"

# ~/.local/bin/cargo-devlog-test
#!/bin/bash
shift
exec devlog-wrapper cargo test "$@"
```

Make sure your wrapper scripts are executable.

For Cargo specifically, this naming convention means you can now
run `cargo devlog-run` and `cargo devlog-test` as Cargo subcommands.

## Usage

1. Navigate to any git repository
2. Run your wrapped command: `cargo devlog-test`
3. Enter a brief description when prompted: "What did you change?"
4. The script runs your command and logs everything

## Data Storage

All devlog data is stored in `.devlog/data/` at the root of your git repository:

```
your-project/
├── .devlog/
│   ├── main/
│   │   ├── 001-entry.toml
│   │   ├── 002-entry.toml
│   │   └── ...
│   └── feature/x11-support/
│       ├── 001-entry.toml
│       └── 002-entry.toml
├── src/
└── Cargo.toml
```

Add `.devlog/` to your `.gitignore` to keep devlog data out of your repositories.
I recommend adding it once to your global .gitignore.

## TOML Entry Format

Each entry contains:

```toml
[metadata]
timestamp = "2025-09-12T10:30:00Z"
sequence = 1
description = "Added basic key event capture for X11"
command = "cargo test"

[git]
commit_hash = "abc123..."
branch = "feature/x11-capture"

[diff]
files_changed = ["src/main.rs", "tests/integration.rs"]
files_created = []
files_deleted = []
diff_content = """
# Full git diff here
"""

[output]
exit_code = 0
content = """
# Command output here
"""
```

## Workflow Example

```bash
# Working on a new feature
$ cargo devlog-test
What are you working on? Fix the key parsing logic for special characters
running 5 tests
test key_parser::test_special_chars ... FAILED
...

# Entry 001-entry.toml created with your commit, diff, and test output

# Try a different approach
$ cargo devlog-run
What did you change? Try using regex instead of manual parsing
Application started successfully
...

# Entry 002-entry.toml created

# When ready, squash commits
```

## License

Licensed under either of the following, at your choice:

- [Apache License, Version 2.0](https://github.com/K-JBoon/devlog-utils/blob/master/LICENSE-APACHE.txt), or
- [MIT license](https://github.com/K-JBoon/devlog-utils/blob/master/LICENSE-MIT.txt)

Unless explicitly stated otherwise, any contribution intentionally submitted
for inclusion in this crate, as defined in the Apache-2.0 license, shall
be dual-licensed as above, without any additional terms or conditions.
