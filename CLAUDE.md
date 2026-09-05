# TW-Base-Fields — contexte projet pour Claude

> **Avant toute tâche sur ce plugin, consulter d'abord le `CLAUDE.md` du workspace** (`../CLAUDE.md`) et ses `guides/` : outillage de dev commun (pnpm, `dev.cjs`/HMR, Ctrl+C, git push), pièges PowerShell/Windows, `publishFilter`, conventions de nommage/i18n, symlink. Ci-dessous : uniquement le spécifique à TW-Base-Fields.

## Ce que c'est
Plugin TiddlyWiki (`$:/plugins/nikorion/base-fields`) qui ajoute 5 champs au template d'édition, en lignes pleine largeur sous la ligne du core qu'elles prolongent. Aucun JS : que du wikitext + CSS. **Aucune surcharge de tiddler core** — tout passe par des sections `$:/tags/EditTemplate` placées avec `list-after`.

## Les 3 lignes ajoutées
Une ligne = un `<div class="bf-extra-row bf-…">` ; chaque contrôle = `<span class="bf-edit-field">` contenant un libellé `<em class="tc-edit tc-small-gap-right">` (transclusion de `Fields/<champ>/Prompt`, `title` = nom réel du champ), le contrôle, puis `bf-delete-field-button`. Exception : l'encadré Outils, `<span class="bf-tools-field">`, n'a pas de libellé (son placeholder en tient lieu).

| Ligne (classe) | Champs | Section (`list-after`) | Macros |
|---|---|---|---|
| `.bf-tags-extras` | `tools` | `extra-fields.tid` (après `…/EditTemplate/tags`) | `bf-tools-field`, `bf-tool-pill` |
| `.bf-project-row` | `project` | idem, juste après la précédente | `bf-project-field` |
| `.bf-type-extras` | `role`, `maturity`, `status` | `vocab-fields.tid` (après `…/EditTemplate/type`) | `bf-select-field` |

Où intervenir : libellé/placeholder → `language/<lang>/fields.multids` ; disposition d'une ligne → `styles/edit-fields.tid` ; ordre des lignes ou déplacement d'un champ → la section correspondante ; comportement d'un contrôle → `macros/edit-fields.tid`.

## Structure
```
src/base-fields/
  ui/EditTemplate/extra-fields.tid  ← lignes Outils puis Projet
  ui/EditTemplate/vocab-fields.tid  ← ligne Rôle + Maturité + Statut
  macros/edit-fields.tid      ← bf-select-field, bf-project-field, bf-tools-field, bf-tool-pill,
                                 bf-delete-field-button, bf-clear-empty-field
  macros/vocab.tid            ← bf-vocab-values / -label / -icon / -hint / -item
  vocab/role.tid              ← valeurs autorisées + ordre, dans le champ `list`
  vocab/maturity.tid
  vocab/status.tid            ← + champ `visible-filter` : quand le contrôle apparaît
  vocab/icons.multids         ← clés `<champ>/<valeur>` → émoji
  vocabularies.tid            ← doc utilisateur des vocabulaires (onglet du plugin + ControlPanel)
  search-filters.tid          ← champs project-search-filter / tools-search-filter
  default-config.multids      ← masque les 4 champs de la liste des champs du core
  language/<lang>/vocab.multids   ← `Vocab/<champ>/<valeur>` + `…/Hint`
  language/<lang>/fields.multids  ← `Fields/<champ>/<Prompt|Placeholder|Delete/Hint|…>`
  styles/edit-fields.tid
```

## Spécificités
- **Ajouter une valeur de vocabulaire** = 3 endroits : champ `list` du `vocab/<champ>.tid`, `vocab/icons.multids`, `language/<lang>/vocab.multids` (deux entrées par langue : `Vocab/<champ>/<valeur>` pour le libellé, `Vocab/<champ>/<valeur>/Hint` pour l'infobulle — la seconde est facultative). Libellé absent → repli sur le slug ; icône absente → rendu sans icône. **Aucune UI de config, volontairement** : une valeur ajoutée depuis l'interface ne pourrait pas emporter ses traductions.
- **Un saut de ligne dans un attribut** (infobulle des options) s'obtient par `[charcode[10]]` : aucune entité HTML ne serait décodée, TW pose la valeur telle quelle via `setAttribute`.
- **`status` n'est affiché que là où il a un sens** — un `role` de `task`/`tasklist` — mais **aussi dès que le champ porte une valeur**, quel que soit le rôle. Sans cette seconde condition, un tiddler passé de tâche à autre chose garderait un `status` invisible dans l'éditeur, toujours matché par les filtres et impossible à effacer. La condition de visibilité vit dans le champ `visible-filter` de `vocab/status.tid`, pas dans la section.
- **Seul le slug est stocké** dans le champ. Icônes et libellés sont résolus au rendu — ne jamais les faire entrer dans la valeur.
- **Chaque `.bf-extra-row` est une ligne pleine largeur**, placée par le seul `list-after` de sa section ; la CSS ne dispose que l'intérieur d'une ligne (flex, `wrap`), rien qui dépende du nesting du formulaire. Ne pas réapparier les blocs avec une ligne du core via une grille sur le conteneur des sections : essayé, abandonné — l'encadré des tags et le bloc `role` partageaient alors une colonne, donc ajouter un tag décalait `role` latéralement, et l'encadré des outils, à l'étroit, empilait ses pastilles au-dessus de sa saisie.
- **Ne pas réintroduire de surcharge core.** Deux tentatives ont été écartées : copier `$:/core/ui/EditTemplate/type` pour y loger Nature/Statut, et copier `.../tags` pour y loger Outils. Une surcharge core fige silencieusement l'ancienne version du tiddler à la montée de version TW (`tags.tid` a d'ailleurs changé entre 5.4.1 et master), et entre en collision avec tout autre plugin touchant la même ligne. Une ligne supplémentaire coûte moins cher.
- **Les saisies outils et projet n'utilisent pas `tag-picker`, et ne doivent pas y revenir.** Cette macro code en dur le placeholder des tags (`$:/language/EditTemplate/Tags/Add/Placeholder`, aucun paramètre pour le changer) et réutilise `newTagNameTiddler`, que `$:/core/ui/EditTemplate` définit une fois pour tout le formulaire : les deux sélecteurs partageraient alors une saisie, et `save-tiddler-actions` (Ctrl+Entrée) ajouterait le texte en cours comme **tag**. Elles pilotent donc `keyboard-driven-input` directement (la macro sur laquelle `tag-picker` et le champ Type du core sont eux-mêmes bâtis), avec leurs propres tiddlers d'état `$:/temp/NewToolName*` et `$:/temp/Project/*`. Les filtres d'autocomplétion vivent sur `search-filters.tid`, atteints via `configTiddlerFilter` + `firstSearchFilterField`.
- **Les filtres de `search-filters.tid` dédupliquent explicitement** (`each:value[]` pour `project`, `each:list-item[tools]` pour `tools`) — ne pas les simplifier : voir le piège de déduplication dans `../CLAUDE.md`.
- **Après avoir écrit un champ que la saisie affiche, poser `refreshTitle` à `yes`** (vu sur le menu Projet) : un `$edit-text` ne touche pas un élément qui a le focus, pour ne pas écraser la frappe — sans ça la saisie continue d'afficher le texte tapé alors que le champ vaut autre chose.
- **`<$select default="">` est obligatoire** sur Nature/Statut : sans cet attribut le widget assigne `undefined` au nœud DOM quand le champ est absent, aucune option ne correspond (`selectedIndex` = -1) et le contrôle s'affiche vide au lieu de retomber sur l'option placeholder. Cette première option vide doit rester : sans elle, un tiddler sans valeur afficherait la première entrée du vocabulaire comme si elle était choisie.
- **Les pastilles d'outils recopient `tag-body-inner`** (cascade `$:/tags/TiddlerColourFilter`, `contrastcolour`, cascade d'icône) : ces procédures sont locales à `$:/core/ui/EditTemplate/tags`, donc inatteignables de l'extérieur. À resynchroniser lors d'une montée de version TW.
- **Un libellé s'aligne sur son contrôle en `align-items: baseline`** (lignes Projet et Rôle/Maturité/Statut), jamais par un `padding-top` : les hauteurs varient d'un contrôle à l'autre — dans l'encadré Outils, la première ligne est tantôt une pastille, tantôt la saisie (champ vide), 7 px d'écart.
- **`project` a sa propre ligne, sous Outils** — pas en bout de la ligne Outils. Il y était (`flex: 0 0 auto`, poussé à droite), mais l'encadré Outils prend toute la largeur qu'on lui donne et le libellé Projet manquait ; deux lignes distinctes se lisent mieux et laissent chacune grandir sans décaler l'autre.
- **Le `$:/config/SyncFilter` du wiki de dev doit exclure les shadows livrés hors du namespace du plugin** (`$:/config/EditTemplateFields/Visibility/`) en plus du préfixe `$:/plugins/nikorion/base-fields/` : sinon un push HMR les fait passer pour des tiddlers modifiés, la synchro les écrit en vrais tiddlers sous `wiki/tiddlers/system/`, et ces copies figées masquent ensuite le plugin. Si ces fichiers réapparaissent : les supprimer, puis vérifier le filtre.
- HMR : tout est `.tid`/`.multids`, donc poussé à chaud dans le navigateur **déjà ouvert** — et **uniquement là**. Un rechargement d'onglet ne montre pas la modif, il l'annule : la page repart de la version chargée au boot du serveur. **Ne jamais conseiller un rechargement d'onglet pour voir une modif** ; soit on regarde l'onglet ouvert sans y toucher, soit on redémarre TW (`touch src/base-fields/plugin.info`, nodemon reboote) avant de recharger.
