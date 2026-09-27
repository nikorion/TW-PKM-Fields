# TW-Base-Fields

Source of the [TiddlyWiki](https://tiddlywiki.com) plugin `$:/plugins/nikorion/base-fields`, which adds ten fields to the tiddler edit template: `role`, `maturity`, `lifecycle`, `needs`, `status`, `project`, `epistemology`, `disciplines`, `technology`, `equipment`. Pure wikitext and CSS, no JavaScript, no core tiddler overridden.

This README is for whoever wants to change the plugin. What the fields are for and how to use them is the plugin's own readme, shown in the wiki (`src/base-fields/language/<lang>/readme.tid`); the demo wiki `docs/TW-Base-Fields-Wiki.html` has it all, with a Playground.

## Getting started

```sh
pnpm install
pnpm dev     # dev wiki (wiki/) + hot reload; the URL (random free port) is printed on start
pnpm build   # dist/TW-Base-Fields-Plugin.json + docs/TW-Base-Fields-Wiki.html
```

`pnpm dev` pushes any edit under `src/base-fields` or `wiki/tiddlers` straight into the browser tab already open; only `plugin.info` restarts the server. **Do not reload the tab to see a change**: the page would come back as the server loaded it at boot, losing what was pushed since. Stop with Ctrl+C twice.

To load the plugin in another Node.js wiki, symlink `src/base-fields` as `$TIDDLYWIKI_PLUGIN_PATH/nikorion/base-fields` and list `"nikorion/base-fields"` in that wiki's `tiddlywiki.info`. Requires TiddlyWiki ≥ 5.3.0.

## Source layout

| Path (under `src/base-fields/`) | Role |
|---|---|
| `ui/EditTemplate/discipline-fields.tid` | row above the tags: `project`, `epistemology`, `disciplines` (`list-before` the core tags) |
| `ui/EditTemplate/extra-fields.tid` | row below the tags: `technology`, `equipment` (`list-after` the core tags) |
| `ui/EditTemplate/vocab-fields.tid` | row below the type: `role`, `maturity`, `lifecycle`, `needs`, `status` |
| `macros/edit-fields.tid` | the controls: `bf-select-field`, `bf-needs-field`, `bf-single-value-field`, `bf-list-value-field`, `bf-list-pill`, delete button |
| `macros/vocab.tid` | global `bf-vocab-*` helpers (values, groups, label, icon, hint) |
| `vocab/<field>.tid` | a vocabulary: `list` (values, in order), `groups`/`group-<slug>` (`role`), `radio`, `blank-value`, `visible-filter` |
| `vocab/icons.multids` | one emoji per `<field>/<value>` |
| `language/<lang>/fields.multids` | prompts, placeholders, tooltips of the controls |
| `language/<lang>/vocab.multids` | value labels `Vocab/<field>/<value>`, optional `…/Hint`, role plurals `…/Plural` |
| `search-filters.tid` | `<field>-search-filter`: completion of each free field |
| `<field>-colour.tid` | fallback pill colour of each list field (text: light palette, `dark` field: dark palette) |
| `default-config.multids` | hides the ten fields from the core field list |
| `styles/edit-fields.tid` | layout inside each row |

## How it works

- **No core override.** Each row is a section tagged `$:/tags/EditTemplate`, placed only by its `list-before`/`list-after`. The stylesheet lays out what is inside a row, never pairs rows with the core ones: rows stay independent, so growing the tags box pushes the next row down and moves nothing sideways. A copied core tiddler would freeze its old version on a TiddlyWiki upgrade and collide with any other plugin touching the same row.
- **Only the slug is stored.** Icons and labels are resolved at render time by the `bf-vocab-*` helpers; never let them into a field value.
- **Generic procedures.** `bf-list-value-field(field)` and `bf-single-value-field(field)` derive everything (state tiddlers, language keys, completion filter) from the field name.
- **No `tag-picker`.** The free fields drive the core `keyboard-driven-input` macro directly: `tag-picker` hardcodes the tags placeholder and shares the form's single tag input, so the save shortcut would add a half-typed value as a *tag*.
- **Visibility.** A vocabulary's `visible-filter` decides on which roles its control shows; a field holding a value always shows, framed in red when the role does not use it.

## Extending

- **A vocabulary value**: its slug in the `list` of `vocab/<field>.tid` (and in a `group-<slug>` for `role`, or it is offered before the first heading), its icon in `vocab/icons.multids`, its label (and optional hint) in each `language/<lang>/vocab.multids`. A new role also needs its `/Plural` in each language. Without a label a value shows its slug; without an icon, no icon.
- **A list or single-value field**: one call to `bf-list-value-field`/`bf-single-value-field` in the row's section, its strings in `language/<lang>/fields.multids`, its `<field>-search-filter`, its line in `default-config.multids`; a list field also needs a `<field>-colour.tid` and a place in `bf-list-placeholder-chars`.
- **Anything used by [TW-Dynamic-Table](https://github.com/nikorion/TW-Dynamic-Table)** (`bf-vocab-*`, `bf-list-pill` and its `readonly`, `bf-list-value-field`, a vocabulary's `blank-value`): its column templates (`src/dyntable/templates/body/`) call them; change them in step, and give a new field its column there (plus its `Tables/Column/<field>` label in each language).

## On a TiddlyWiki upgrade

`bf-list-pill` copies the core's `tag-body-inner` (colour and icon cascades, `contrastcolour`), a procedure local to `$:/core/ui/EditTemplate/tags` and so unreachable from outside: diff it against the new core and resync.

## License

MIT — see `LICENSE`.
