# TW-Base-Fields

Source of the [TiddlyWiki](https://tiddlywiki.com) plugin `$:/plugins/nikorion/base-fields`, the editor of the *kms* suite: it puts the fields defined by [TW-KMS-Ontology](https://github.com/nikorion/TW-KMS-Ontology) in the tiddler edit template. Pure wikitext and CSS, no JavaScript, no core tiddler overridden.

This README is for whoever wants to change the plugin. How the editor behaves for a wiki user is the plugin's own readme (`src/base-fields/language/<lang>/readme.tid`); what the fields mean, and their vocabularies, belong to the ontology. The demo wiki `docs/TW-Base-Fields-Wiki.html` has it all, with a Playground.

## Getting started

```sh
pnpm install
pnpm dev     # dev wiki (wiki/) + hot reload; the URL (random free port) is printed on start
pnpm build   # dist/TW-Base-Fields-Plugin.json + docs/TW-Base-Fields-Wiki.html
```

`pnpm dev` pushes any edit under `src/base-fields` or `wiki/tiddlers` straight into the browser tab already open; only `plugin.info` restarts the server. **Do not reload the tab to see a change**: the page would come back as the server loaded it at boot, losing what was pushed since. Stop with Ctrl+C twice. An edit to the ontology is not pushed by this wiki: restart it, or work in the suite's integration wiki (`../KMS`), which watches every plugin of the suite.

To load the plugin in another Node.js wiki, symlink `src/base-fields` as `$TIDDLYWIKI_PLUGIN_PATH/nikorion/base-fields` (and the ontology as `…/nikorion/kms-ontology`) and list both in that wiki's `tiddlywiki.info`. Requires TiddlyWiki ≥ 5.3.0 and TW-KMS-Ontology.

## Source layout

| Path (under `src/base-fields/`) | Role |
|---|---|
| `ui/EditTemplate/discipline-fields.tid` | row above the tags (`list-before` the core tags) |
| `ui/EditTemplate/extra-fields.tid` | row below the tags (`list-after` the core tags), then the free fields no row names |
| `ui/EditTemplate/vocab-fields.tid` | row below the type, then the vocabulary fields no row names |
| `rows.multids` | which fields each row shows, in order (`$:/config/nikorion/base-fields/row/<row>`) |
| `controls.multids` | vocabulary fields drawn as radio buttons instead of a dropdown (`…/control/<field>: radio`) |
| `macros/edit-fields.tid` | `bf-field` (draws a field according to its `kind`) and one control per kind: `bf-select-field`, `bf-checkbox-field`, `bf-single-value-field`, `bf-list-value-field`; delete button, editor strings (`bf-field-text`) |
| `search-filters.tid` | completion filters of the free fields (`value-search-filter`, `list-search-filter`) |
| `language/<lang>/fields.multids` | the editor's own strings: generic (`Fields/<key>`) and per field when the wording needs it (`Fields/<field>/<key>`) |
| `readme/controls.tid` | the readme's table of controls, generated |
| `default-config.multids` | hides the fields from the core field list |
| `compat/` | deprecated: `bf-vocab-*` aliases and the icon titles of 0.3, kept so existing wikitext and `icon` fields go on working |
| `styles/edit-fields.tid` | layout inside each row |

## How it works

- **The ontology decides, the editor draws.** Nothing here knows a field by name: `bf-field` reads the field's `kind` (`vocab`, `vocab-list`, `value`, `list`), its label, description, vocabulary and `applies-filter` through the ontology API (`kms-*` functions), and draws the matching control. A field added to the ontology gets a control with no change here.
- **No core override.** Each row is a section tagged `$:/tags/EditTemplate`, placed only by its `list-before`/`list-after`. The stylesheet lays out what is inside a row, never pairs rows with the core ones: rows stay independent, so growing the tags box pushes the next row down and moves nothing sideways. A copied core tiddler would freeze its old version on a TiddlyWiki upgrade and collide with any other plugin touching the same row.
- **No `tag-picker`.** The free fields drive the core `keyboard-driven-input` macro directly: `tag-picker` hardcodes the tags placeholder and shares the form's single tag input, so the save shortcut would add a half-typed value as a *tag*.
- **Visibility.** A field shows where its `applies-filter` accepts the tiddler; one that does not apply but holds a value shows all the same, framed in red.

## Extending

- **A field or a vocabulary value** is added to the ontology (see its README), not here. A new field lands at the end of the type row (vocabulary) or of the row under the tags (free) until `rows.multids` places it; give it a line in `default-config.multids` so the core field list does not show it twice, and, if the generic editor strings read badly for it, its own `Fields/<field>/…` strings.
- **A new kind** needs a control procedure in `macros/edit-fields.tid` and a branch in `bf-field-control`, plus its line in `language/<lang>/readme.multids` (`Readme/Control/<kind>`).
- **Anything [TW-Dynamic-Table](https://github.com/nikorion/TW-Dynamic-Table) calls** — `bf-list-value-field(field)`, used to edit its list columns: change its signature in step with `src/dyntable/procedures/dt-kms.tid`.

## License

MIT — see `LICENSE`.
