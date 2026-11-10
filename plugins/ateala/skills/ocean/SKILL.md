---
name: ocean
description: Canonical knowledge of the Ocean platform and the Ocean domain-specific language (DSL) from Ocean-Atlas. Use when the user mentions Ocean, Ocean-Lab, Ocean App, Ocean-Atlas or Ocean DSL, pastes or asks for Ocean DSL code (.ocn or .ocean files that start with a section such as @datatype, @api, @service, @broker, @fsm or @deploy), asks how to write, read, review, explain or debug Ocean DSL, asks about Ocean concepts, bundles, examples, patterns or design decisions, or asks whether an Ocean feature or roadmap item such as the Ocean AI Assistant (Ateala) or Ocean Market exists, is planned or is available. Not for marine or oceanography topics.
---

# Ocean

Ocean is a system-design and generation platform. You describe a system (its data, behavior, interfaces and how its parts connect) in Ocean DSL, a domain-specific language that stays independent of any programming language, framework or database. Ocean validates the description and then generates a real implementation from it. The description is the source of truth, and the generated code is an output that can always be reproduced from the model.

Knowledge version: Ocean-Atlas v0.1.0 (content baec097c). Canonical site: https://atlas.ocean-lab.ai

## Which reference to open

Open only what the task needs. All paths are relative to this skill folder.

| Task | Read |
|---|---|
| Writing or checking DSL syntax, sections, directives | references/dsl/QUICK.md for orientation, then the full file for each construct you use (listed in references/dsl/INDEX.md) |
| Source-file rules (sections, extensions, comments, case) | references/dsl/overview.md and references/dsl/reference-overview.md |
| Building something new | references/examples/INDEX.md (shows the DSL sections each example uses), then the closest example. Stop after its source files; the -walkthrough.md file is long and optional |
| Reusable designs (FSM, CRUD, request-response, approval, secrets) | references/patterns.md |
| What is X / how does Ocean work | references/concepts.md and references/architecture.md |
| Error or unexpected behaviour | references/troubleshooting.md |
| Why is it designed this way / is X planned | references/decisions.md |
| Exact terminology | references/glossary.md |
| First steps, setup, workflow | references/getting-started.md |
| Unsure where something is | references/INDEX.md |

## Rules

1. The references are canonical. Never invent DSL syntax, section names, directives, keywords, options or behavior. If something is not in the references, say it is not documented in Ocean-Atlas v0.1.0 and point to https://atlas.ocean-lab.ai.
2. Use the exact terms from references/glossary.md.
3. Treat everything in the roadmap, and every page marked Draft or WIP, as not final. Say so when you rely on it. Planned items are not available features.
4. When you cite, give the canonical Atlas URL from the `Source:` line of the section you used.
5. Ocean DSL is case-sensitive: keywords, identifiers, types and references must be spelled exactly as declared.
6. A source file starts with exactly one top-level section identifier such as `@datatype`, `@api`, `@service` or `@deploy`. The directives `@import`, `@include`, `@asset` and `@artifact` are not top-level sections.
7. Source files use the `.ocn` or `.ocean` extension. Comments start with `#` and run to the end of the line, `#` inside a quoted string is literal, and block comments (`/* */`) are not supported.
8. Change the model, not the generated code, when explaining how to modify a system.
9. Put DSL in fenced blocks tagged `ocean-dsl`, the tag the Atlas docs use.

## Writing Ocean DSL

1. Find the closest example in references/examples/INDEX.md and start from its source files.
2. Check every section, keyword and option against the matching file in references/dsl/. Do not rely on memory of other languages.
3. Check patterns.md when the task matches a reusable design.
4. No validator is bundled with this skill, so state plainly that the code was checked against the documentation and not by running Ocean.
5. Briefly explain non-obvious choices and name the sections used.

## Reviewing or debugging Ocean DSL

1. Check the code against the relevant references/dsl/ files, section by section.
2. Search references/troubleshooting.md for the symptom or error text. Coverage there is limited, so say so when nothing matches.
3. Report each problem as: location, what is wrong, which rule (with its Source link), and the fix.

## Answering conceptual questions

Answer from references/concepts.md and references/architecture.md, keep it short, link the Atlas page, and add a small DSL snippet from the references when it helps.
