# Usage

Prompt Composer expands reusable text in native Kilo agent prompts, command templates and skill-tool bodies. It requires Node 22.18+ and targets Kilo 7.8.27 and later 7.8.x releases. It works independently of Config Composer.

## Install

Add the package to `plugin` in `kilo.jsonc`, and to `tui.jsonc` for the prompt viewer:

```jsonc
{ "plugin": ["@lunchbox-labs/kilo-prompt-composer"] }
```

## Include reusable text

Put snippets in a `prompts/` directory, such as `.kilo/prompts/review/findings.md`. Include them at the desired position in an agent, command or skill body:

```markdown
Review the supplied changes.

{{include:review/findings.md}}

Distinguish observed behavior from assumptions.
```

References are relative `.md` or `.txt` filenames. Includes expand recursively against the same discovered prompt registry; each included file is trimmed at its edges. Surrounding text determines spacing. Write `\{{include:review/findings.md}}` to preserve a literal marker. [Discovery](Discovery) describes directory precedence, supported paths and limits.

Use `{{include?:personal/review.md}}` for optional guidance. If no discovered `prompts/` directory contains that file, the marker expands to nothing. For example, a developer can opt in by adding `~/.config/kilo/prompts/personal/review.md`. All other behavior matches `{{include:...}}`: an invalid path, an unreadable or invalid file, a cycle, or the include count or depth limit leaves that include out with a warning. Write `\{{include?:personal/review.md}}` to preserve the literal optional marker.

Keep all guidance in native bodies and snippet files. There is no Prompt Composer settings file, and plugin options are ignored with a warning. Agent metadata and command placeholders such as `$ARGUMENTS` and `$1` remain native. Disabled agents and agents with no nonblank authored prompt keep their existing text or native fallback.

Skill bodies expand when the native `skill` tool returns them. Automatically generated skill slash commands do not pass through that hook and do not expand includes; running one that contains include markers shows a warning once per skill. Skill discovery and permissions remain Kilo's responsibility.

## Snippet lifetime

The plugin captures the project's winning snippet files when it first initializes in the server process, including snippets first used by later skill calls. Edits, additions and removals are visible after restarting that process. Instance disposal alone does not refresh snippets, except that a capture that failed is retried. Restart the Kilo server after editing snippets or native prompt definitions.

Include problems never stop a prompt from loading. A failing include expands to nothing, a malformed marker stays as written, and a prompt over the 256 KiB limit, or one with nothing left once failed includes are left out, keeps its native text; other agents, commands and skills compose normally. A warning toast names the consumer and shows the first problem with its logical include chain and, for nested snippets, the snippet position; the Kilo log lists every problem. If the prompts directories cannot be read at all, an error toast appears and everything keeps its native text. Diagnostics omit prompt bodies.

## View loaded prompts

Run `/view-prompts`, choose **Agents** or **Commands**, then select a component to read its full loaded text in a scrollable viewer. Text over 65,536 characters, or with a line over 4,096 characters, opens as a list of parts. Escape returns to the previous view. Configured agents appear by default; **Show built-in agents** includes native defaults, and the host remembers that choice. Disabled agents and native fallbacks are labeled.

The viewer shows loaded text without executing command syntax, expanding markers again, changing files or sending a model request. It omits skills because the host API does not expose their composed bodies for this view. It does not show the full model request.

## Includes and native file references

Kilo substitutes `{file:path}` inside JSON configuration strings before plugins run. The path resolves relative to the configuration file or `~/`; the reference is not recursive, has no escape, and a missing file fails configuration loading. Use it to load one whole prompt from a file.

Includes also work inside Markdown agent, command and skill bodies. They expand recursively, resolve against discovered `prompts/` directories and accept the `\{{include:...}}` escape. A prompt loaded through `{file:}` can contain includes because the plugin receives the substituted text. Optional includes have no `{file:}` equivalent.

## Examples

- [Review guidance](https://github.com/lunchbox-labs/kilo-prompt-composer/tree/main/docs/examples/review) shares a findings format between an agent and command.
- [On-demand verification](https://github.com/lunchbox-labs/kilo-prompt-composer/tree/main/docs/examples/skill-guidance) loads a checklist through a skill.
- [Migration](Migration) covers older Prompt Composer settings and optional Config Composer ordering.
