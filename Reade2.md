# Contribution-README
# Contribution [2]: Support for command line options

**Contribution Number:** [2]  
**Student:** Paul Tsekpo  
**Issue:** [https://github.com/dosbox-staging/dosbox-staging/issues/2183](https://github.com/session-foundation/session-desktop/issues/839)  
**Status:** Round 2 [Phase I] [Completed]

---

## Why I Chose This Issue

I chose this issue because I wanted to gain experience with the loguru logging module and understand how large open-source projects handle configurable logging. It was also a well-scoped, self-contained feature that was a good fit for a first contribution.

---

## Understanding the Issue

### Problem Description

There was no option to log to files once the executable started up. The only option available to the user was to log to the console. In other words, the loguru functionality that supports logging to a file was never wired up through the config system.

### Expected Behavior

Users should be able to choose between four logging destinations via the config file:
- `console` — log to the command line interface only
- `file` — log to a rolling log file on disk only
- `console-and-file` — log to both simultaneously (default)
- `off` — disable all logging

Log files should be named `dosbox-staging-NNN.log` where NNN is a zero-padded index, stored in a `logs` subdirectory of the DOSBox config directory. A maximum of 5 log files should be kept, with the oldest deleted automatically when the limit is exceeded.

### Current Behavior

The only option available to the user was to log to the console. There were no config settings for log destination or log file path.

### Affected Components

- `src/dosbox.cpp` — main implementation: config registration, rolling log helpers, and log setup on startup

---

## Reproduction Process

### Environment Setup

- Cloned the repository locally in VS Code on Windows
- Installed vcpkg dependencies and set the `VCPKG_ROOT` environment variable
- Built the project using the CMake preset for `release-windows-vs2022` to verify the baseline built cleanly before making any changes

### Steps to Reproduce

1. In VS Code, trigger a build — this produces the executable at `./build/release-windows-vs2022/Release/dosbox-staging.exe`
2. Run the executable
3. Observe that all logs go to the console only, with no config option to redirect to a file

### Reproduction Evidence

- **Fork:** https://github.com/blackheart-5/dosbox-staging
- **My findings:** Before this change, logs were always sent to the console. There was no `log_destination` or `log_path` config setting in the `[dosbox]` section.

---

## Solution Approach

### Analysis

The config system in DOSBox Staging is well-established — other settings like `machine`, `hard_disk_speed`, and `vesa_modes` follow a clear pattern of registering a string option in `add_dosbox_config_section` and reading it in `dosbox_realinit`. The loguru library already supports file logging via `loguru::add_file`; it just needed to be wired up through the config system. The additional requirement for rolling logs meant the file setup needed helper functions to scan the log directory, determine the next index, and prune old files.

### Proposed Solution

Add two new config settings to the `[dosbox]` section:
- `log_destination` — enum string: `console`, `file`, `console-and-file`, `off`
- `log_path` — path to the directory where log files are stored (defaults to `<config_dir>/logs`)

Implement rolling log support with three focused helper functions:
1. `collect_log_files` — scans the log directory and returns a sorted map of index → path for all matching `dosbox-staging-NNN.log` files
2. `prune_old_logs` — deletes the oldest files from that map until fewer than `MAX_LOG_FILES` (5) exist
3. `next_log_path` — orchestrates the above and returns the full path for the new log file

### Implementation Plan

1. Register `log_destination` and `log_path` string/path config settings in `add_dosbox_config_section`
2. Implement `collect_log_files` to iterate the log directory, validate filenames against the `dosbox-staging-` prefix and `.log` suffix, parse the numeric index, and return a sorted `std::map<int32_t, std::filesystem::path>`
3. Implement `prune_old_logs` to delete oldest entries from that map until `size < MAX_LOG_FILES`
4. Implement `next_log_path` to call collect → prune → compute next index → return the assembled file path
5. In `dosbox_realinit`, read `log_destination` and branch: set up `loguru::add_file` with the path from `next_log_path` for file/console-and-file modes, suppress stderr for file-only mode

**Implement:** https://github.com/dosbox-staging/dosbox-staging/compare/main...blackheart-5:dosbox-staging:main

**Review:** Follows `CONTRIBUTING.md` conventions and conventional commit message format

**Evaluate:** After building and running, confirm a `logs/` folder is created in the config directory containing a `dosbox-staging-001.log` file on first run, `002` on the second, and so on, with the oldest file deleted once 5 are present

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: `collect_log_files` returns empty map for an empty directory
- [ ] Test case 2: `collect_log_files` correctly ignores files that don't match the `dosbox-staging-NNN.log` pattern
- [ ] Test case 3: `next_log_path` returns index 1 when no existing log files are present
- [ ] Test case 4: `next_log_path` returns max existing index + 1 when files are present
- [ ] Test case 5: `prune_old_logs` deletes oldest files until fewer than 5 remain

### Integration Tests

- [ ] Run the executable 6 times with `log_destination = file` and confirm only 5 log files exist after the 6th run, with the lowest-indexed file deleted
- [ ] Confirm `log_destination = console-and-file` produces both a log file on disk and output on stderr

### Manual Testing

- Built and ran the executable with `log_destination = console-and-file` (default) — confirmed a `logs/` directory was created in the DOSBox config folder containing `dosbox-staging-001.log`
- Ran again — confirmed `dosbox-staging-002.log` was created and `001.log` still present
- Ran 4 more times past the 5-file limit — confirmed the oldest file was deleted and the count stayed at 5
- Tested `log_destination = file` — confirmed stderr produced no log output
- Tested `log_destination = off` — confirmed no log output anywhere

---

## Implementation Notes

### Week 1 Progress

Explored the codebase to understand the existing config and logging setup. Identified that loguru was already vendored and partially used but not wired to a config setting. Studied how other `[dosbox]` section settings like `hard_disk_speed` are registered and read to follow the same pattern.

### Week 2 Progress

Implemented the initial version of `find_next_log_index` and the call-site branching in `dosbox_realinit`. Encountered several build errors on Windows (MSVC via VS2022 Build Tools) including typos (`std::stoic` → `std::stoi`, `start_with` → `starts_with`), incorrect use of `loguru::FileMode` as a callable, and a missing-return code path in `collect_log_files`. Fixed all build errors and resolved SonarCloud quality gate failures including catching specific exceptions, using `std::format` instead of `snprintf`, and scoping variables with C++17 init-statements.

### Week 3 Progress

Addressed PR review feedback: removed accidentally committed `.claude/settings.local.json` from the PR and added it to `.gitignore`. Refactored the implementation to follow single-responsibility principle — split the original combined function into `collect_log_files`, `prune_old_logs`, and `next_log_path`.

### Code Changes

- **Files modified:** `src/dosbox.cpp`
- **Files added to `.gitignore`:** `.claude/settings.local.json`
- **Key commits:** https://github.com/dosbox-staging/dosbox-staging/compare/main...blackheart-5:dosbox-staging:main
- **Approach decisions:** Chose `std::map<int32_t, std::filesystem::path>` keyed by log index so the map is automatically sorted, making both the maximum (for next index) and minimum (for pruning) trivially accessible via `rbegin()` and `begin()` respectively. Used `std::filesystem::directory_iterator` with a `std::error_code` overload rather than exceptions to handle inaccessible directories gracefully.

---

## Pull Request

**PR Link:** https://github.com/dosbox-staging/dosbox-staging/pull/4930

**PR Description:**

> Implements rolling log file support via two new `[dosbox]` config options: `log_destination` (console / file / console-and-file / off) and `log_path` (directory for log files, defaults to `<config_dir>/logs`). Log files are named `dosbox-staging-NNN.log` with a zero-padded running index. On startup the log directory is scanned, the next index is determined, and any files beyond the 5-file limit are pruned oldest-first.

**Maintainer Feedback:**

- [Review]: Flagged accidentally committed `.claude/settings.local.json` — removed from PR and added to `.gitignore`
- [Review]: SonarCloud Quality Gate failures addressed — missing return path in `collect_log_files`, overly generic exception catch, `snprintf` replaced with `std::format`, C++17 init-statements used for locally-scoped variables

**Status:** Iterating on review feedback

---

## Learnings & Reflections

### Technical Skills Gained

- How to use `std::filesystem::directory_iterator` with error codes for graceful failure handling
- How `std::map` sorting can be exploited structurally (min/max accessible via `begin()`/`rbegin()`) rather than needing a separate sort step
- How `std::optional` works and the distinction between calling methods on the optional itself vs. the dereferenced value (`->` vs `.`)
- How `std::format` (C++20) replaces `snprintf` for type-safe, buffer-overflow-safe string formatting
- How C++17 if-init-statements scope variables tightly to their usage context
- How loguru's `add_file` API works and how `FileMode::Truncate` vs `Append` differ

### Challenges Overcome

- Multiple MSVC build errors that didn't appear in other compilers — particularly the `loguru::FileMode` callable error and the `optional` member access errors — required carefully reading error messages to distinguish typos from real design bugs
- SonarCloud's Quality Gate catching a missing return path in `collect_log_files` that the compiler did not warn about — the `return` statements were accidentally placed inside the `for` loop body, meaning an empty directory caused the function to fall through with no return value
- Accidentally committing a personal Claude Code config file with hardcoded local paths — learned to check `.gitignore` before committing IDE/tool config files

### What I'd Do Differently Next Time

- Set up `.gitignore` entries for tool-specific local config files before starting work, not after they've already been committed to a PR
- Run SonarCloud or a local static analyzer earlier in the process rather than waiting for CI to catch issues
- Write unit tests in parallel with the implementation rather than leaving them to the end

---

## Resources Used

- [loguru documentation](https://github.com/emilk/loguru)
- [std::filesystem reference — cppreference.com](https://en.cppreference.com/w/cpp/filesystem)
- [std::optional reference — cppreference.com](https://en.cppreference.com/w/cpp/utility/optional)
- [std::format reference — cppreference.com](https://en.cppreference.com/w/cpp/utility/format/format)
- [DOSBox Staging CONTRIBUTING.md](https://github.com/dosbox-staging/dosbox-staging/blob/main/CONTRIBUTING.md)
- [GitHub Issue #2183](https://github.com/dosbox-staging/dosbox-staging/issues/2183)
- [Pull Request #4930](https://github.com/dosbox-staging/dosbox-staging/pull/4930)
