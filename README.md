# TW-PKM-Fields

**English** · [Français](README.fr.md)

Source of the [TiddlyWiki](https://tiddlywiki.com) plugin `$:/plugins/nikorion/pkm-fields`, the interface of the *pkm* suite: it puts the fields defined by [TW-PKM-Schema](https://github.com/nikorion/TW-PKM-Schema) in the tiddler edit template, and in the columns of [TW-Dynamic-Table](https://github.com/nikorion/TW-Dynamic-Table) when that plugin is installed. Pure wikitext and CSS, no JavaScript, no core tiddler overridden.

This README is for whoever wants to change the plugin. How the editor behaves for a wiki user is the plugin's own readme (`src/pkm-fields/language/<lang>/readme.tid`); what the fields mean, and their vocabularies, belong to the schema. The demo wiki `docs/TW-PKM-Fields-Wiki.html` has it all, with a Playground.

## Contents

- [Getting started](#getting-started)
- [Source layout](#source-layout)
- [How it works](#how-it-works)
- [Extending](#extending)
- [License](#license)

## Getting started

```sh
pnpm install
pnpm dev     # dev wiki (wiki/) + hot reload; the URL (random free port) is printed on start
pnpm build   # dist/TW-PKM-Fields-Plugin.json + docs/TW-PKM-Fields-Wiki.html
```

`pnpm dev` pushes any edit under `src/pkm-fields` or `wiki/tiddlers` straight into the browser tab already open; only `plugin.info` restarts the server. **Do not reload the tab to see a change**: the page would come back as the server loaded it at boot, losing what was pushed since. Stop with Ctrl+C twice. An edit to the schema is not pushed by this wiki: restart it, or work in the suite's integration wiki (`../PKM`), which watches every plugin of the suite.

To load the plugin in another Node.js wiki, symlink `src/pkm-fields` as `$TIDDLYWIKI_PLUGIN_PATH/nikorion/pkm-fields` (and the schema as `…/nikorion/pkm-schema`) and list both in that wiki's `tiddlywiki.info`. Requires TiddlyWiki ≥ 5.3.0 and TW-PKM-Schema.

[↑ Back to contents](#contents)

## Source layout

The source is in three layers, each in its own folder. `controls/` draws and writes a field's value; `editor/` and `dyntable/` place those controls, each in its own dress. The dependency goes one way only: `editor/` and `dyntable/` call `controls/`, never each other, and `controls/` calls neither — only the schema API.

| Path (under `src/pkm-fields/`) | Role |
|---|---|
| `controls/<kind>.tid` | one bare control per kind — `nk-vocab-select` / `nk-vocab-radios` (`vocab`), `nk-vocab-checkboxes` (`vocab-list`), `nk-value-input` (`value`), `nk-list-input` (`list`), `nk-date-input` (`date`) — writing the field of `currentTiddler`; no label, no delete button |
| `controls/common.tid` | what the controls share: the plugin's strings (`nk-field-text`, `nk-placeholder`), the actions run when a field changes (clear an empty value, `icon` following the role), why a field is flagged (`nk-stale-hint`) |
| `controls/search-filters.tid` | completion filters of the free fields (`value-search-filter`, `list-search-filter`) |
| `controls/styles.tid` | the controls' own styles |
| `editor/fields.tid` | `nk-field`: places a field's control in the editor according to its `kind`, with a label, a delete button and a red frame; row contents (`nk-row-fields`) |
| `editor/EditTemplate/discipline-fields.tid` | row above the tags (`list-before` the core tags) |
| `editor/EditTemplate/extra-fields.tid` | row below the tags (`list-after` the core tags), then the free fields no row names |
| `editor/EditTemplate/vocab-fields.tid` | row below the type, then the vocabulary fields no row names |
| `editor/styles.tid` | layout inside each row |
| `rows.multids` | which fields each row shows, in order (`$:/config/nikorion/pkm-fields/row/<row>`) |
| `controls.multids` | vocabulary fields drawn as radio buttons instead of a dropdown in the editor (`…/control/<field>: radio`) |
| `language/<lang>/fields.multids` | the plugin's own strings: generic (`Fields/<key>`) and per field when the wording needs it (`Fields/<field>/<key>`) |
| `readme/controls.tid` | the readme's table of controls, generated |
| `default-config.multids` | hides the fields from the core field list |
| `dyntable/body/<kind>.tid` | Dynamic Table cell templates, one per kind (`vocab`, `vocab-list`, `value`, `list`, `date`), picked by their `nk-dyntable-column-filter` |
| `dyntable/procedures.tid` | what those templates call (`nk-pkm-*`: the value in view mode, the bare control in edit mode), imported into every table (`$:/tags/nk-dyntable/Procedure`) |
| `dyntable/column-label.tid`, `dyntable/row-tones.tid` | a field's label as column header; a record's tones (`pkm-tones`) as row classes `nk-dyntable-row-<tone>` |
| `dyntable/styles.tid` | the table cells' own styles |

[↑ Back to contents](#contents)

## How it works

- **The schema decides, the editor draws.** Nothing here knows a field by name: `nk-field` reads the field's `kind` (`vocab`, `vocab-list`, `value`, `list`), its label, description, vocabulary and `applies-filter` through the schema API (`pkm-*` functions), and draws the matching control. A field added to the schema gets a control with no change here.
- **No core override.** Each row is a section tagged `$:/tags/EditTemplate`, placed only by its `list-before`/`list-after`. The stylesheet lays out what is inside a row, never pairs rows with the core ones: rows stay independent, so growing the tags box pushes the next row down and moves nothing sideways. A copied core tiddler would freeze its old version on a TiddlyWiki upgrade and collide with any other plugin touching the same row.
- **No `tag-picker`.** The free fields drive the core `keyboard-driven-input` macro directly: `tag-picker` hardcodes the tags placeholder and shares the form's single tag input, so the save shortcut would add a half-typed value as a *tag*.
- **Visibility.** A field shows where its `applies-filter` accepts the tiddler; one that does not apply but holds a value shows all the same, framed in red.
- **Tables through Dynamic Table's extension points.** Dynamic Table knows nothing of the schema: the `dyntable/` tiddlers plug into it by its tags (cell templates, column labels, row classes, procedures), so they stay inert when it is absent, and it stays usable without the pkm suite. They rely on the names it hands a template — its README, § Extending from another plugin, is that contract.

[↑ Back to contents](#contents)

## Extending

- **A field or a vocabulary value** is added to the schema (see its README), not here. A new field lands at the end of the type row (vocabulary) or of the row under the tags (free) until `rows.multids` places it; give it a line in `default-config.multids` so the core field list does not show it twice, and, if the generic editor strings read badly for it, its own `Fields/<field>/…` strings.
- **A new kind** needs a control in `controls/<kind>.tid`, a branch placing it in `nk-field-control` (`editor/fields.tid`), its line in `language/<lang>/readme.multids` (`Readme/Control/<kind>`), and a table cell template `dyntable/body/<kind>.tid` (until then Dynamic Table shows the raw value).
- **A tone** (see the schema's API) needs no change here: it reaches a table row as `nk-dyntable-row-<tone>`; style that class if Dynamic Table does not (it styles `success` and `danger`).

[↑ Back to contents](#contents)

## License

MIT — see `LICENSE`.

[↑ Back to contents](#contents)
