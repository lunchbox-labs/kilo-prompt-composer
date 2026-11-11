# Developer documentation

For contributors and maintainers working on Prompt Composer:

- [Local builds and checks](#build-a-local-package): local builds, runtime boundaries and compatibility checks.
- [Publishing](Publishing): trusted publisher setup and the release workflow.

The plugin's state machines are documented in [docs/state-machines.md](https://github.com/lunchbox-labs/kilo-prompt-composer/blob/main/docs/state-machines.md). These wiki pages are published from [docs/wiki](https://github.com/lunchbox-labs/kilo-prompt-composer/tree/main/docs/wiki) on every push to `main` that changes them; change them there.

The repository's [AGENTS.md](https://github.com/lunchbox-labs/kilo-prompt-composer/blob/main/AGENTS.md) and linked shared standards remain the authoritative machine instructions. Runnable examples and the license stay in the repository and published package. [User documentation](Home#user-documentation) covers installation and everyday use.

Prompt Composer's include safety engine and dialog navigation originated in the MIT-licensed [Config Composer](https://github.com/lunchbox-labs/opencode-config-composer) project. Configuration composition belongs to Config Composer, and inline prompt inclusion to Prompt Composer.

## Build a local package

Use the committed npm lockfile. Development and CI use Node 26.4 or later to match the Kilo TUI build dependencies. The published server and include APIs require Node 22.18 or later; Kilo renders the TUI with its bundled Bun runtime.

```sh
npm ci
npm run build
npm pack
```

The package is marked private while local testing and release setup are pending; `npm pack` still works.

Install the generated tarball into a separate directory. Register that installed package's absolute directory in `kilo.jsonc` and, when testing the viewer, `tui.jsonc`. Kilo selects the corresponding server or TUI export. This exercises a local candidate without publishing it.

## Runtime boundaries

The server composes native agent and command bodies through the config hook and skill bodies through the skill tool's after hook. It preserves host metadata and publishes no partially composed config. The TUI reads loaded config and agent metadata; it does not write files or execute prompts.

Startup snippet capture is shared by normalized project context for the lifetime of the imported module. Instance disposal can rerun plugin factories without resetting that capture; a fresh process loads changed snippets. The separately exported `includes` API reads files when called. Keep these distinct lifetime contracts when changing the implementation.

Composition degrades instead of failing: a failing include expands to nothing, a malformed marker stays literal, and a consumer over the 256 KiB limit or left empty keeps its native text, each with a warning toast and a server log entry. Past the include count limit, only the first limit problem is reported. Consumers compose independently, hooks do not reject, and discovery used by startup capture skips unusable entries. A failed capture is not cached, so the next plugin factory captures again. The exported `includes` API and discovery without a skip callback keep their strict behavior.

Kilo's built-in skills use `dir: builtin` and a shorter wrapper; filesystem skills retain their sampled-file footer. Includes are expanded after Kilo's native shell processing, so shell syntax inside included snippets remains literal.

The skill parser isolates the body from native framing and leaves names, directories and sampled filenames literal. When include markers are present, truncated output, unknown or ambiguous framing, or missing or invalid native name and directory metadata leaves the native output unchanged so the skill still loads; a warning toast explains that its includes were not expanded. Exact wrapper strings in tests protect this host boundary.

## Checks

| Command | Coverage |
| --- | --- |
| `npm run check` | Typechecking, lint, formatting and unit tests |
| `npm run test:package` | Built tarball installation, public exports, runtime use and declarations |
| `npm run test:native` | Real Kilo server, npm plugin loading, skill execution, process restart and terminal viewer |

The native suite uses isolated configuration, disables telemetry, and allows only its local fixture provider for both primary and auxiliary models. No provider account is needed. It downloads the host and verifies the ripgrep prerequisite. Some viewer checks replace rendering, while the PTY/ConPTY path exercises real terminal interaction and asserts that browsing sends no model request. Preserve both layers.

Tests should protect observable behavior, safety boundaries and host contracts. Use existing representative paths instead of duplicating wording or increasing test counts. The [shared standards](https://github.com/lunchbox-labs/.github/blob/main/agent-guidance/common.md#meaningful-tests) define the testing policy.

## Host compatibility

The authoritative range and floor live in [tests/kilo.json](https://github.com/lunchbox-labs/kilo-prompt-composer/blob/main/tests/kilo.json); they are separate from pinned SDK/build dependencies and are not shipped in the package. No Kilo runtime engine restriction is declared.

By default, native tests fetch fresh canonical `@kilocode/cli` metadata, choose the newest stable version in that range and install the exact platform binary. They verify the actual CLI version and fail on registry or artifact errors instead of using stale fallback versions. Set `KILO_TEST_VERSION=floor` for the configured floor, or an exact version inside the range.

[Check CI](https://github.com/lunchbox-labs/kilo-prompt-composer/blob/main/.github/workflows/check.yml) covers Linux and Windows and latest-compatible and floor Kilo hosts. Config Composer's OpenCode package is not part of the Kilo compatibility baseline. The optional `CONFIG_COMPOSER_DIR` harness is reserved for a separately ported package; do not use an OpenCode package as evidence of Kilo compatibility.
