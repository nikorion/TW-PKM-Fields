# TW-Base-Fields

A [TiddlyWiki](https://tiddlywiki.com) plugin adding four everyday fields to the tiddler edit template.

| Field | Control | Stored value |
|---|---|---|
| `nature` | controlled vocabulary, single value | a bare slug (`concept`, `guide`, …) |
| `status` | controlled vocabulary, single value | a bare slug (`stable`, `raw`, …) |
| `project` | free text, with a dropdown of the values already used in the wiki | free text |
| `tools` | list field, edited like tags, on the Tags row | a list of titles |

`nature`, `status` and `project` share the **Type** row: the four controls sit on one flex line and shrink evenly — Type included — so the row always fits the tiddler width. `tools` shares the **Tags** row, as a second box of the same kind — the two pickers stay strictly independent: the tools dropdown offers the values already used in `tools`, never the wiki's tags, and vice versa.

Only the slug is ever stored. Icons and labels (en-GB, fr-FR) are display-only and resolved at render time, so a wiki stays readable and filterable in any language.

## Customising the vocabularies

There is no vocabulary editor in the UI, on purpose: a value added through the interface could not carry its translations. Everything is in the source:

- allowed values and their order — the `list` field of `src/base-fields/vocab/nature.tid` and `vocab/status.tid`
- icons — `src/base-fields/vocab/icons.multids`, keyed `<field>/<value>`
- labels — `src/base-fields/language/<lang>/vocab.multids`, keyed `Vocab/<field>/<value>`

A value with no label falls back to its own slug; a value with no icon renders without one.

## Development

```sh
pnpm install
pnpm dev     # dev wiki on http://localhost:8080, with HMR
pnpm build   # dist/TW-Base-Fields-Plugin.json + docs/TW-Base-Fields-Wiki.html
```

## Note

The plugin ships verbatim copies of two core tiddlers, `$:/core/ui/EditTemplate/type` and `$:/core/ui/EditTemplate/tags`, with the extra controls inserted inside them — that is the only way to share their flex row. Upgrading TiddlyWiki will not bring in upstream changes to those two tiddlers until this plugin is updated too.

## License

MIT — see `LICENSE`.
