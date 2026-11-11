# Kilo Prompt Composer

Share reusable prompt text across native Kilo agents, commands and on-demand skills with explicit inline includes.

Requires Node **22.18+**. Targets Kilo **7.8.27** and later **7.8.x** releases. This fork is not published yet; [build a local package](docs/wiki/Developer.md#build-a-local-package) to test it.

## Install

Add the package to `plugin` in `kilo.jsonc`, and to `tui.jsonc` for the prompt viewer:

```jsonc
{ "plugin": ["@lunchbox-labs/kilo-prompt-composer"] }
```

## Include shared text

Write shared text in `.kilo/prompts/review.md`:

```markdown
Check the implementation and report actionable findings with evidence.
```

Include it where it belongs in an agent prompt, command template or skill body:

```markdown
Review the supplied changes.

{{include:review.md}}

Distinguish observed behavior from assumptions.
```

Use `{{include?:personal/review.md}}` for an include that may be absent, and run `/view-prompts` to read the loaded agent and command text. Restart the Kilo server after editing snippets. A problem with an include shows a warning toast and never stops a prompt from loading; [Usage](docs/wiki/Usage.md#snippet-lifetime) describes what is kept.

## Documentation

The [guides](docs/wiki/Home.md) cover:

- [Usage](docs/wiki/Usage.md): includes, optional includes, the prompt viewer and native file references.
- [Discovery](docs/wiki/Discovery.md): prompt directories, precedence and limits.
- [Migration](docs/wiki/Migration.md): older Prompt Composer settings and Config Composer ordering.
- [Developer](docs/wiki/Developer.md): development, compatibility testing and publishing.

[Review guidance](docs/examples/review/) and [On-demand verification](docs/examples/skill-guidance/) are runnable example projects, each with a walkthrough.
