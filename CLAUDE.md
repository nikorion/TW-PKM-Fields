# TW-Base-Fields — contexte projet pour Claude

> **Avant toute tâche sur ce plugin, consulter d'abord le `CLAUDE.md` du workspace** (`../CLAUDE.md`) et ses `guides/` : outillage de dev commun (pnpm, `dev.cjs`/HMR, Ctrl+C, git push), pièges PowerShell/Windows, `publishFilter`, conventions de nommage/i18n, symlink. Ci-dessous : uniquement le spécifique à TW-Base-Fields.

## Ce que c'est
Plugin TiddlyWiki (`$:/plugins/nikorion/base-fields`) qui ajoute 4 champs au template d'édition : `nature`, `status` (vocabulaires contrôlés, valeur unique), `project` (texte libre + menu des valeurs déjà utilisées), `tools` (champ liste, édité comme les tags). Aucun JS : que du wikitext + CSS.

## Structure
```
src/base-fields/
  ui/EditTemplate/type.tid    ← SEULE SURCHARGE CORE (copie conforme + Nature/Statut)
  ui/EditTemplate/tools.tid   ← section propre, list-after: $:/core/ui/EditTemplate/tags
  ui/EditTemplate/project.tid ← section propre, list-after: .../tools
  macros/edit-fields.tid      ← bf-select-field, bf-project-field, bf-tools-field, bf-tool-pill,
                                 bf-delete-field-button, bf-clear-empty-field
  macros/vocab.tid            ← bf-vocab-values / -label / -icon / -hint / -item / -option
  vocab/nature.tid            ← valeurs autorisées + ordre, dans le champ `list`
  vocab/status.tid
  vocab/icons.multids         ← clés `<champ>/<valeur>` → émoji
  search-filters.tid          ← champs project-search-filter / tools-search-filter
  default-config.multids      ← masque les 4 champs de la liste des champs du core
  language/<lang>/vocab.multids   ← `Vocab/<champ>/<valeur>` + `…/Hint`
  language/<lang>/fields.multids  ← `Fields/<champ>/<Prompt|Placeholder|Delete/Hint|…>`
  styles/edit-fields.tid
```

## Spécificités
- **Ajouter une valeur de vocabulaire** = 3 endroits : champ `list` du `vocab/<champ>.tid`, `vocab/icons.multids`, `language/<lang>/vocab.multids` (deux entrées par langue : `Vocab/<champ>/<valeur>` pour le libellé, `Vocab/<champ>/<valeur>/Hint` pour l'infobulle — la seconde est facultative). Libellé absent → repli sur le slug ; icône absente → rendu sans icône. **Aucune UI de config, volontairement** : une valeur ajoutée depuis l'interface ne pourrait pas emporter ses traductions.
- **Seul le slug est stocké** dans le champ. Icônes et libellés sont résolus au rendu — ne jamais les faire entrer dans la valeur.
- **`ui/EditTemplate/type.tid` est la seule surcharge core.** La régénérer depuis le clone `TiddlyWiki5` **à la version ciblée** (`git -C <clone> show v5.4.1:core/ui/EditTemplate/type.tid`, retirer les CR) plutôt que l'éditer à l'aveugle, puis réinsérer le bloc `<!-- Base Fields -->` avant le `</div>` qui ferme `tc-edit-type-selector-wrapper`. **Avant de monter la `core-version`, lire le diff amont de ce fichier** (`git diff v<ancienne> v<nouvelle> -- core/ui/EditTemplate/type.tid`) — sinon la surcharge annule silencieusement les évolutions core. Ne jamais partir du working tree du clone sans vérifier sa version : il est sur `master` (5.5.0-prerelease), en avance sur le 5.4.1 installé.
- **Outils et Projet ne surchargent rien, et ne doivent pas le faire.** Ils partagent la ligne des tags par la CSS : `.tc-tiddler-edit-frame > .tc-keyboard > .tc-keyboard` (le conteneur des sections) passe en grille 3 colonnes, chaque section s'étalant sur `1 / -1` sauf `.tc-edit-tags`, `.bf-tools-field` et `.bf-edit-field-project`. Un précédent essai fusionnait Outils dans une copie de `$:/core/ui/EditTemplate/tags` : écarté — ce fichier change entre versions TW (`tag-body-inner` réécrit en amont), et la panne aurait été silencieuse. Ici, si le nesting `.tc-keyboard` change, les encadrés se réempilent : visible, inoffensif.
- **Les saisies outils et projet n'utilisent pas `tag-picker`, et ne doivent pas y revenir.** Cette macro code en dur le placeholder des tags (`$:/language/EditTemplate/Tags/Add/Placeholder`, aucun paramètre pour le changer) et réutilise `newTagNameTiddler`, que `$:/core/ui/EditTemplate` définit une fois pour tout le formulaire : les deux sélecteurs partageraient alors une saisie, et `save-tiddler-actions` (Ctrl+Entrée) ajouterait le texte en cours comme **tag**. `bf-tools-field` pilote donc `keyboard-driven-input` directement (la macro sur laquelle `tag-picker` et le champ Type du core sont eux-mêmes bâtis) avec ses propres tiddlers d'état `$:/temp/NewToolName*`. Les filtres d'autocomplétion vivent sur `search-filters.tid` (champs `tools-search-filter` et `project-search-filter`), atteints via `configTiddlerFilter` + `firstSearchFilterField`.
- **Après avoir écrit un champ que la saisie affiche, poser `refreshTitle` à `yes`** (vu sur le menu Projet) : un `$edit-text` ne touche pas un élément qui a le focus, pour ne pas écraser la frappe — sans ça la saisie continue d'afficher le texte tapé alors que le champ vaut autre chose.
- **Les pastilles d'outils recopient `tag-body-inner`** (cascade `$:/tags/TiddlerColourFilter`, `contrastcolour`, cascade d'icône) : ces procédures sont locales à `$:/core/ui/EditTemplate/tags`, donc inatteignables de l'extérieur. À resynchroniser lors d'une montée de version TW.
- **`<$select default="">` est obligatoire** sur Nature/Statut : sans cet attribut le widget assigne `undefined` au nœud DOM quand le champ est absent, aucune option ne correspond (`selectedIndex` = -1) et le contrôle s'affiche vide au lieu de retomber sur l'option placeholder. Cette première option vide doit rester : sans elle, un tiddler sans valeur afficherait la première entrée du vocabulaire comme si elle était choisie.
- **Le libellé Outils s'aligne en `align-items: baseline`**, pas par un `padding-top` : la première ligne de l'encadré est tantôt une pastille, tantôt la saisie (champ vide), et leurs hauteurs diffèrent de 7 px.
- La CSS neutralise le `min-width` core calculé par `set-type-selector-min-width` : sans ça le contrôle de type refuse de rétrécir. Plancher `10em` par contrôle → passage à la ligne au lieu d'un écrasement illisible.
- **Le `$:/config/SyncFilter` du wiki de dev doit exclure les shadows surchargés par le plugin** (`$:/core/ui/EditTemplate/type`, `$:/config/EditTemplateFields/Visibility/`) en plus du préfixe `$:/plugins/nikorion/base-fields/` : sinon un push HMR les fait passer pour des tiddlers modifiés, la synchro les écrit en vrais tiddlers sous `wiki/tiddlers/system/`, et ces copies figées masquent ensuite le plugin. Si ces fichiers réapparaissent : les supprimer, puis vérifier le filtre.
- HMR : tout est `.tid`/`.multids`, donc poussé à chaud dans le navigateur **déjà ouvert** ; un rechargement de page repart de la version chargée au boot du serveur. Pour valider une modif après rechargement : `touch src/base-fields/plugin.info` (nodemon redémarre TW).
