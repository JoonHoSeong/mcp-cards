---
name: doca-argp
category: devops
description: Use when implementing a standardized command-line interface (CLI) for DOCA applications using the doca-argp library, including parameter registration, value callbacks, and JSON-based configuration.
trigger: "User wants to add new CLI flags to a DOCA sample, implement JSON-config files for their application, or troubleshoot why a CLI flag is not being parsed correctly."
---

# DOCA Arg Parser (doca-argp) Skill Card

## Overview
`doca-argp` is a utility library that standardizes the command-line interface (CLI) for all DOCA applications. It provides a consistent way to define parameters (flags), handle their values via callbacks, and allow the entire application configuration to be driven by a JSON file.

### Core Value Proposition
- **Standardized Surface**: Ensures all DOCA apps have a consistent look and feel (e.g., `--device`, `--json`, `--help`).
- **Callback-Driven Logic**: Decouples the parsing of the command line from the application logic via value callbacks.
- **JSON-Config Bridge**: Allows complex configurations to be offloaded to a JSON file, which is parsed using the same registration logic as the CLI flags.
- **Automatic Documentation**: Generates the `--help` output automatically based on the registered parameters.

## Implementation Path: The Arg Parser Lifecycle

### 1. Integration & Registration (`## configure`)
1. **Presence Check**: Confirm `doca-argp` is installed using `pkg-config --exists doca-argp`.
2. **Registration Phase**: Call `doca_argp_register_param` for every new flag. This must happen **before** calling `doca_argp_start`.
3. **Callback Definition**: Define a callback function that specifies how the parsed value should be written into the application's configuration struct.
4. **Standard Surface Audit**: Ensure the application calls `doca_argp_init` to inherit the standard DOCA CLI flags.

### 2. Execution & Parsing (`## run`)
1. **Activation**: Call `doca_argp_start(argc, argv)` to trigger the parsing process.
2. **Help Generation**: Use the built-in `--help` flag to verify that all registered parameters and their descriptions are correctly listed.
3. **Value Retrieval**: Access the configuration struct that was populated by the registered callbacks.
4. **JSON-Config Drive**: Use the `--json <path>` flag to load configuration from a file. Confirm the JSON keys match the long names of the registered parameters.

### 3. Evaluation & Validation (`## test`)
**The "CLI-Coverage" Loop.**
1. **Help Smoke**: Verify every new flag appears in the `--help` output.
2. **Argv Smoke**: Pass the new flag via the command line and confirm the callback fires and updates the config struct.
3. **Regression Test**: Run the application with only the standard flags to ensure existing behavior is not broken.
4. **JSON-Config Smoke**: Drive the same configuration via a JSON file and confirm the behavior is identical to the argv-driven run.

### 4. Triage & Debugging (`## debug`)
**The "Arg-Parser-Failure" ladder.**
1. **Lifecycle Order**: If `doca_argp_start` returns `BAD_STATE`, check if `doca_argp_start` was called twice or if a parameter was registered after the parser started.
2. **Type Mismatch**: If a flag returns `INVALID_VALUE`, verify that the value passed matches the registered parameter type (e.g., string vs. integer).
3. **JSON Key Mismatch**: If `--json` returns `NOT_SUPPORTED`, diff the JSON keys against the registered long names of the parameters.
4. **IO Failures**: If `--json` returns `IO_FAILED`, verify the file exists, is readable, and contains valid JSON (using `jq`).
5. **Standard Surface Override**: If a standard flag (like `--device`) stopped working, check if it was accidentally re-registered with a different callback.

## Critical Rules & Safety

### 1. The "Register-Before-Start" Mandate
**All `doca_argp_register_param` calls must be completed before `doca_argp_start` is invoked.**
- **Rule**: Registering after start results in `BAD_STATE` and the flag will not be parsed.

### 2. The "Standard-Surface" Protection
**Do not re-register flags owned by `doca_argp_init`.**
- **Rule**: Standard flags like `--device` and `--representor` are managed by the library; attempting to re-register them can break default behavior.

### 3. The "JSON-Key" Exactness
**JSON keys must exactly match the long names of the registered parameters.**
- **Rule**: Even a small typo in the JSON key will result in `NOT_SUPPORTED` during the parse.

### 4. The "Callback-Purity" Rule
**Callbacks should only update the configuration struct; they should not trigger complex application logic.**
- **Rule**: Keep callbacks lightweight to ensure the parsing phase remains fast and predictable.

## Command Appendix

### Arg Parser Invocations
Use structured helpers first. Fall back to manual commands if probes fail.

| Purpose | Command | Healthy Output |
| --- | --- | --- |
| Installation Check | `pkg-config --exists doca-argp && echo ok` | `ok`. |
| Build Version | `pkg-config --modversion doca-argp` | Semver string matching `doca_caps --version`. |
| Linker Flags | `pkg-config --cflags --libs doca-argp` | Correct include and link flags for the install. |
| Sample Audit | `ls /opt/mellanox/doca/samples/*/*_main.c` | List of sample mains for identifying CLI modification targets. |
| Help Verification | `"$BINARY" --help` | Listing of all standard and user-added flags. |
| JSON Validation | `jq . -- "$JSON_PATH"` | Valid, pretty-printed JSON tree. |
| Trace Logging | `DOCA_LOG_LEVEL=trace "$BINARY" --json "$JSON_PATH"` | Trace-level lines for every parameter parse call. |

## Deferred Topic Boundaries
- **General DOCA Lifecycle**: Route to `doca-common` for context and memory management.
- **General Debugging**: Route to `doca-debug` for system-wide triage.
- **Environment Setup**: Route to `doca-setup` for driver and library installation.
- **Version Management**: Route to `doca-version` for detailed version matrix lookups.
