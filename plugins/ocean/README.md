# Ocean plugin for Claude Code

Gives Claude the canonical Ocean platform and Ocean DSL knowledge from [Ocean-Atlas](https://atlas.ocean-lab.ai), so it can write, review, explain, and debug Ocean DSL accurately.

## Install

```text
/plugin marketplace add gilmardcom/ocean-claude-plugins
/plugin install ocean@ocean-lab
```

The skill is then available as `/ocean:ocean`, and Claude also uses it automatically when a conversation involves Ocean.

For claude.ai, Claude desktop, the API, and manual installation, see [Use Ocean with Claude](https://atlas.ocean-lab.ai/getting-started/use-with-claude/).

## Generated content

`skills/ocean/` and `.claude-plugin/plugin.json` are **generated** from the Ocean-Atlas knowledge by `builders/claude-skill`. Do not edit it by hand. After changing canonical docs, run:

```bash
npm --prefix builders/claude-skill run build:skill
```

The skill contains documentation only. It runs no scripts and fetches nothing from the network.
