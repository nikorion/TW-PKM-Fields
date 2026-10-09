# TW-PKM-Fields

[English](README.md) · **Français**

Sources du plugin [TiddlyWiki](https://tiddlywiki.com) `$:/plugins/nikorion/pkm-fields`, l'interface de la suite *pkm* : il place les champs définis par [TW-PKM-Schema](https://github.com/nikorion/TW-PKM-Schema) dans le modèle d'édition des tiddlers, et dans les colonnes de [TW-Dynamic-Table](https://github.com/nikorion/TW-Dynamic-Table) quand ce plugin est installé. Du wikitext et du CSS purs, sans JavaScript, sans aucun tiddler du core surchargé.

Ce README s'adresse à qui veut modifier le plugin. Le comportement de l'éditeur pour un utilisateur du wiki relève du readme du plugin lui-même (`src/pkm-fields/language/<lang>/readme.tid`) ; le sens des champs et leurs vocabulaires relèvent du schéma. La [démo en ligne](https://nikorion.github.io/TW-PKM-Fields/) montre tout cela, avec un Playground.

## Prise en main

```sh
pnpm install
pnpm dev     # wiki de dev (wiki/) + rechargement à chaud ; l'URL (port libre aléatoire) s'affiche au démarrage
pnpm build   # dist/TW-PKM-Fields-Plugin.json + docs/ (wiki de démo, publié par la CI)
```

`pnpm dev` pousse toute modification sous `src/pkm-fields` ou `wiki/tiddlers` directement dans l'onglet de navigateur déjà ouvert ; seul `plugin.info` redémarre le serveur. **Ne pas recharger l'onglet pour voir une modification** : la page reviendrait telle que le serveur l'a chargée au démarrage, en perdant ce qui a été poussé depuis. Arrêter avec deux Ctrl+C. Une modification du schéma n'est pas poussée par ce wiki : le redémarrer, ou travailler dans le wiki d'intégration de la suite (`../PKM`), qui surveille tous les plugins de la suite.

Pour charger le plugin dans un autre wiki Node.js, créer un lien symbolique de `src/pkm-fields` vers `$TIDDLYWIKI_PLUGIN_PATH/nikorion/pkm-fields` (et du schéma vers `…/nikorion/pkm-schema`) et ajouter les deux au `tiddlywiki.info` de ce wiki. Requiert TiddlyWiki ≥ 5.3.0 et TW-PKM-Schema.

## Organisation des sources

Les sources se répartissent en trois couches, chacune dans son dossier. `controls/` dessine et écrit la valeur d'un champ ; `editor/` et `dyntable/` placent ces contrôles, chacun à sa façon. La dépendance va dans un seul sens : `editor/` et `dyntable/` appellent `controls/`, jamais l'un l'autre, et `controls/` n'appelle ni l'un ni l'autre — seulement l'API du schéma.

| Chemin (sous `src/pkm-fields/`) | Rôle |
|---|---|
| `controls/<kind>.tid` | un contrôle nu par nature — `nk-vocab-select` / `nk-vocab-radios` (`vocab`), `nk-vocab-checkboxes` (`vocab-list`), `nk-value-input` (`value`), `nk-list-input` (`list`), `nk-date-input` (`date`) — qui écrit le champ de `currentTiddler` ; ni libellé, ni bouton de suppression |
| `controls/common.tid` | ce que partagent les contrôles : les chaînes du plugin (`nk-field-text`, `nk-placeholder`), les actions exécutées quand un champ change (vider une valeur vide, `icon` qui suit le rôle), la raison pour laquelle un champ est signalé (`nk-stale-hint`) |
| `controls/search-filters.tid` | filtres de complétion des champs libres (`value-search-filter`, `list-search-filter`) |
| `controls/styles.tid` | les styles propres aux contrôles |
| `editor/fields.tid` | `nk-field` : place le contrôle d'un champ dans l'éditeur selon son `kind`, avec un libellé, un bouton de suppression et un cadre rouge ; contenu des rangées (`nk-row-fields`) |
| `editor/EditTemplate/discipline-fields.tid` | rangée au-dessus des tags (`list-before` les tags du core) |
| `editor/EditTemplate/extra-fields.tid` | rangée sous les tags (`list-after` les tags du core), puis les champs libres qu'aucune rangée ne nomme |
| `editor/EditTemplate/vocab-fields.tid` | rangée sous le type, puis les champs de vocabulaire qu'aucune rangée ne nomme |
| `editor/styles.tid` | disposition à l'intérieur de chaque rangée |
| `rows.multids` | quels champs chaque rangée affiche, dans l'ordre (`$:/config/nikorion/pkm-fields/row/<row>`) |
| `controls.multids` | champs de vocabulaire dessinés en boutons radio plutôt qu'en liste déroulante dans l'éditeur (`…/control/<field>: radio`) |
| `language/<lang>/fields.multids` | les chaînes propres au plugin : génériques (`Fields/<key>`) et par champ quand la formulation l'exige (`Fields/<field>/<key>`) |
| `readme/controls.tid` | le tableau des contrôles du readme, généré |
| `default-config.multids` | masque les champs dans la liste des champs du core |
| `dyntable/body/<kind>.tid` | modèles de cellule de Dynamic Table, un par nature (`vocab`, `vocab-list`, `value`, `list`, `date`), choisis par leur `nk-dyntable-column-filter` |
| `dyntable/procedures.tid` | ce qu'appellent ces modèles (`nk-pkm-*` : la valeur en mode lecture, le contrôle nu en mode édition), importé dans chaque tableau (`$:/tags/nk-dyntable/Procedure`) |
| `dyntable/column-label.tid`, `dyntable/row-tones.tid` | le libellé d'un champ comme en-tête de colonne ; les tonalités d'un enregistrement (`pkm-tones`) comme classes de ligne `nk-dyntable-row-<tone>` |
| `dyntable/styles.tid` | les styles propres aux cellules du tableau |

## Fonctionnement

- **Le schéma décide, l'éditeur dessine.** Rien ici ne connaît un champ par son nom : `nk-field` lit le `kind` du champ (`vocab`, `vocab-list`, `value`, `list`), son libellé, sa description, son vocabulaire et son `applies-filter` via l'API du schéma (fonctions `pkm-*`), et dessine le contrôle correspondant. Un champ ajouté au schéma reçoit un contrôle sans aucune modification ici.
- **Aucune surcharge du core.** Chaque rangée est une section taguée `$:/tags/EditTemplate`, placée uniquement par son `list-before`/`list-after`. La feuille de style dispose ce qui se trouve dans une rangée, sans jamais apparier les rangées avec celles du core : les rangées restent indépendantes, si bien qu'agrandir la boîte des tags pousse la rangée suivante vers le bas sans rien décaler latéralement. Un tiddler du core recopié figerait son ancienne version lors d'une mise à jour de TiddlyWiki et entrerait en conflit avec tout autre plugin touchant la même rangée.
- **Pas de `tag-picker`.** Les champs libres pilotent directement la macro `keyboard-driven-input` du core : `tag-picker` code en dur le texte indicatif des tags et partage l'unique saisie de tag du formulaire, si bien que le raccourci d'enregistrement ajouterait une valeur à moitié tapée comme *tag*.
- **Visibilité.** Un champ s'affiche là où son `applies-filter` accepte le tiddler ; un champ qui ne s'applique pas mais contient une valeur s'affiche tout de même, encadré de rouge.
- **Les tableaux via les points d'extension de Dynamic Table.** Dynamic Table ignore tout du schéma : les tiddlers de `dyntable/` s'y branchent par ses tags (modèles de cellule, libellés de colonne, classes de ligne, procédures), si bien qu'ils restent inertes en son absence, et qu'il reste utilisable sans la suite pkm. Ils s'appuient sur les noms qu'il fournit à un modèle — son README, § Extending from another plugin, constitue ce contrat.

## Extension

- **Un champ ou une valeur de vocabulaire** s'ajoute au schéma (voir son README), pas ici. Un nouveau champ se place à la fin de la rangée du type (vocabulaire) ou de la rangée sous les tags (libre) jusqu'à ce que `rows.multids` le positionne ; lui donner une ligne dans `default-config.multids` pour que la liste des champs du core ne l'affiche pas en double et, si les chaînes génériques de l'éditeur sonnent mal pour lui, ses propres chaînes `Fields/<field>/…`.
- **Une nouvelle nature** exige un contrôle dans `controls/<kind>.tid`, une branche qui le place dans `nk-field-control` (`editor/fields.tid`), sa ligne dans `language/<lang>/readme.multids` (`Readme/Control/<kind>`), et un modèle de cellule de tableau `dyntable/body/<kind>.tid` (d'ici là, Dynamic Table affiche la valeur brute).
- **Une tonalité** (voir l'API du schéma) n'exige aucune modification ici : elle atteint une ligne de tableau sous la forme `nk-dyntable-row-<tone>` ; styler cette classe si Dynamic Table ne le fait pas (il style `success` et `danger`).

## Installation

**Démo en ligne** : [https://nikorion.github.io/TW-PKM-Fields/](https://nikorion.github.io/TW-PKM-Fields/) — pour essayer le plugin avant de l'installer.

**Depuis la bibliothèque de plugins nikorion** (TiddlyWiki propose ensuite chaque nouvelle version en mise à jour) :

1. Sur [nikorion.github.io/tw-plugins](https://nikorion.github.io/tw-plugins/), glisser le bouton **Bibliothèque de plugins nikorion** sur votre wiki (une fois par wiki).
2. Ouvrir *Panneau de configuration → Plugins → Obtenir d'autres plugins → Ouvrir la bibliothèque de plugins*, choisir l'onglet nikorion et installer **PKM Fields**.

**À la main** : télécharger [`TW-PKM-Fields-Plugin.json`](https://nikorion.github.io/TW-PKM-Fields/TW-PKM-Fields-Plugin.json) et le glisser-déposer sur votre wiki.

Nécessite TiddlyWiki ≥ 5.3.0.

## Licence

MIT — voir `LICENSE`.
