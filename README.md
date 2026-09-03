# TW-Base-Fields

A [TiddlyWiki](https://tiddlywiki.com) plugin adding four everyday fields to the tiddler edit template.

| Field | Row | Control | Stored value |
|---|---|---|---|
| `tools` | under Tags | list field, edited like tags | a list of titles |
| `project` | under Tags | single value, typed or picked | free text |
| `nature` | under Type | controlled vocabulary, single value | a bare slug (`concept`, `guide`, …) |
| `status` | under Type | controlled vocabulary, single value | a bare slug (`stable`, `raw`, …) |

The four controls come in two rows added to the edit template as ordinary sections — Tools and Project under the tags row, Nature and Status under the type row. **No core tiddler is overridden.** Each row packs its blocks to the left at their natural width, and wraps when the tiddler gets too narrow.

Only the slug is ever stored for `nature` and `status`. Icons and labels (en-GB, fr-FR) are display-only and resolved at render time, so a wiki stays readable and filterable in any language.

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

Everything is a section tagged `$:/tags/EditTemplate` plus a stylesheet: nothing overrides a core tiddler, so a TiddlyWiki upgrade cannot silently revert part of the editor.

## License

MIT — see `LICENSE`.
