# TW-Base-Fields

A [TiddlyWiki](https://tiddlywiki.com) plugin adding four everyday fields to the tiddler edit template.

| Field | Row | Control | Stored value |
|---|---|---|---|
| `tools` | Tags | list field, edited like tags | a list of titles |
| `project` | Tags | single value, typed or picked | free text |
| `role` | Type | controlled vocabulary, single value | a bare slug (`concept`, `guide`, …) |
| `maturity` | Type | controlled vocabulary, single value | a bare slug (`stable`, `raw`, …) |
| `status` | Type | controlled vocabulary, shown on task roles only | a bare slug (`todo`, `doing`, …) |

The controls join the core rows — Tools and Project beside the tags box, Role, Maturity and Status beside the type control — with **no core tiddler overridden**: the pairing is done in the stylesheet, by turning the container the edit-template sections share into a two-column grid. Each pair packs its blocks to the left at their natural width, and drops onto a row of its own on a narrow window.

A *slug* is the short, lowercase, unpunctuated identifier that stands for a value — `snippet`, `raw`. The word comes from the printing trade: a slug was a line of text cast in a single piece of metal, and newsrooms came to call the short working name written on a story in production its slug; web publishing kept it for the identifier that stands for something in a URL, or here in a field.

Only the slug is ever stored for `role`, `maturity` and `status`. Icons and labels (en-GB, fr-FR) are display-only and resolved at render time, so a wiki stays readable and filterable in any language.

## Customising the vocabularies

There is no vocabulary editor in the UI, on purpose: a value added through the interface could not carry its translations. Everything is in the source:

- allowed values and their order — the `list` field of `src/base-fields/vocab/role.tid`, `vocab/maturity.tid` and `vocab/status.tid`
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

Everything is a section tagged `$:/tags/EditTemplate` plus a stylesheet: nothing overrides a core tiddler, so a TiddlyWiki upgrade cannot silently revert part of the editor. The stylesheet does lean on the edit form's internal nesting to pair the rows; should that change, the sections simply stack again.

## License

MIT — see `LICENSE`.
