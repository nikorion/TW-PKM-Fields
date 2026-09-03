# TW-Base-Fields — contexte projet pour Claude

> **Avant toute tâche sur ce plugin, consulter d'abord le `CLAUDE.md` du workspace** (`../CLAUDE.md`) et ses `guides/` : outillage de dev commun (pnpm, `dev.cjs`/HMR, Ctrl+C, git push), pièges PowerShell/Windows, `publishFilter`, conventions de nommage/i18n, symlink. Ci-dessous : uniquement le spécifique à TW-Base-Fields.

## Ce que c'est
Plugin TiddlyWiki (`$:/plugins/nikorion/base-fields`) qui ajoute 4 champs au template d'édition : `nature`, `status` (vocabulaires contrôlés, valeur unique), `project` (texte libre + menu des valeurs déjà utilisées), `tools` (champ liste, édité comme les tags). Aucun JS : que du wikitext + CSS.

## Structure
```
src/base-fields/
  ui/EditTemplate/type.tid    ← SURCHARGE CORE (copie conforme + Nature/Statut/Projet)
  ui/EditTemplate/tags.tid    ← SURCHARGE CORE (copie conforme + encadré Outils sur la ligne Tags)
  macros/edit-fields.tid      ← bf-select-field, bf-project-field, bf-tools-field, bf-clear-empty-field
  macros/vocab.tid            ← bf-vocab-values / -label / -icon / -item
  vocab/nature.tid            ← valeurs autorisées + ordre, dans le champ `list`
  vocab/status.tid
  vocab/icons.multids         ← clés `<champ>/<valeur>` → émoji
  language/<lang>/vocab.multids   ← clés `Vocab/<champ>/<valeur>` → libellé
  language/<lang>/fields.multids  ← clés `Fields/<champ>/<Prompt|None|...>`
  styles/edit-fields.tid
```

## Spécificités
- **Ajouter une valeur de vocabulaire** = 3 endroits : champ `list` du `vocab/<champ>.tid`, `vocab/icons.multids`, `language/<lang>/vocab.multids` (une entrée par langue). Libellé absent → repli sur le slug ; icône absente → rendu sans icône. **Aucune UI de config, volontairement** : une valeur ajoutée depuis l'interface ne pourrait pas emporter ses traductions.
- **Seul le slug est stocké** dans le champ. Icônes et libellés sont résolus au rendu — ne jamais les faire entrer dans la valeur.
- **`ui/EditTemplate/type.tid` et `ui/EditTemplate/tags.tid` sont des surcharges core.** Les régénérer depuis le clone `TiddlyWiki5` **à la version ciblée** (`git -C <clone> show v5.4.1:core/ui/EditTemplate/<f>.tid | tr -d CR`) plutôt que les éditer à l'aveugle, puis réinsérer le bloc `<!-- Base Fields -->`. Attention : `tags.tid` **diffère entre 5.4.1 et master** (`tag-body-inner` réécrit) — ne jamais partir du working tree du clone sans vérifier. **Avant de monter la `core-version`, lire le diff amont des deux fichiers** (`git diff v<ancienne> v<nouvelle> -- core/ui/EditTemplate/type.tid core/ui/EditTemplate/tags.tid`) — sinon la surcharge annule silencieusement les évolutions core.
- **Le sélecteur d'outils masque `newTagNameTiddler` / `newTagNameSelectionTiddler`** (`$:/temp/NewToolName…`) : `$:/core/ui/EditTemplate` ne les définit qu'une fois pour tout le formulaire, donc sans ce masquage les sélecteurs tags et outils partagent une saisie, et `save-tiddler-actions` (Ctrl+Entrée) ajoute le texte en cours comme **tag**. Ne pas retirer ce `$let`.
- La CSS neutralise le `min-width` core calculé par `set-type-selector-min-width` : sans ça le contrôle de type refuse de rétrécir. Plancher `10em` par contrôle → passage à la ligne au lieu d'un écrasement illisible.
- HMR : tout est `.tid`/`.multids`, donc poussé à chaud dans le navigateur **déjà ouvert** ; un rechargement de page repart de la version chargée au boot du serveur. Pour valider une modif après rechargement : `touch src/base-fields/plugin.info` (nodemon redémarre TW).
