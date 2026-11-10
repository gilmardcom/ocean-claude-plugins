# Ateala: Ocean knowledge for Claude

Ateala gives Claude the canonical Ocean platform and Ocean DSL knowledge from [Ocean-Atlas](https://atlas.ocean-lab.ai), so it can write, review, explain, and debug Ocean DSL accurately instead of guessing.

Ocean is a system-design and generation platform: you describe a system in Ocean DSL, and Ocean validates the description and generates an implementation from it. This plugin contains one skill, `ocean`, that Claude uses whenever a conversation involves Ocean or Ocean DSL.

## Install

```text
/plugin marketplace add gilmardcom/ocean-claude-plugins
/plugin install ateala@ocean-lab
```

The skill is then available as `/ateala:ocean`, and Claude also uses it automatically. For claude.ai, Claude desktop, the API, and manual installation, see [Use Ocean with Claude](https://atlas.ocean-lab.ai/getting-started/use-with-claude/).

## Example prompts

- "Write an Ocean DSL definition for an invoice with an id, an amount, and a status enum."
- "How do comments work in Ocean DSL?"
- "Which Ocean example is closest to a broker-based request-response service?"
- "Review this Ocean `@service` and tell me what is wrong with it."
- "Is the Ocean AI Assistant available yet?"

## How it works

The skill is documentation only. It contains `SKILL.md` (routing and rules) and a `references/` folder: one file per DSL section, complete examples with all DSL source, concepts, patterns, a glossary, decisions, and the roadmap. Claude opens only the files a question needs.

It has no scripts, hooks, or MCP servers. It makes no network requests and sends no data anywhere. Every cited fact links back to a canonical Atlas page.

## Limits and troubleshooting

- **No validator.** The skill cannot run Ocean. Claude checks the DSL it writes against the documentation and says so; confirm important code in Ocean App.
- **Documented behavior only.** If something is not in Ocean-Atlas, Claude should say it is not documented. Pages marked Draft or WIP, and everything on the roadmap, are not final or not yet available.
- **The skill does not seem to be used.** Start a fresh conversation after installing, mention "Ocean DSL" or "Ocean-Atlas" in the prompt, or invoke `/ateala:ocean` directly.
- **Out-of-date answers.** Update the plugin (`claude plugin update ateala@ocean-lab`), or download the latest zip from the page above.
- **Report problems with the content** at https://atlas.ocean-lab.ai.

## Generated content

`skills/ocean/` and `.claude-plugin/plugin.json` are **generated** from the Ocean-Atlas knowledge by `builders/claude-skill`. Do not edit them by hand.

See `LICENSE` for the terms of use.
