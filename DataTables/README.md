# Tables n8n

Les CSV contiennent uniquement les en-têtes d’origine, sans aucune ligne de collection. Créez les tables dans votre projet n8n, puis sélectionnez-les dans tous les nœuds Data Table concernés.

L’import CSV permet de reprendre les noms des colonnes. Sans données, vérifiez les types ci-dessous ; si votre version refuse un CSV vide, créez ces colonnes manuellement. Les colonnes système de n8n (`id`, `createdAt`, `updatedAt`) ne sont pas à ajouter aux modèles.

## Audible

| Colonne | Type |
|---|---|
| `id_unique` | string |
| `titre` | string |
| `auteur` | string |
| `duree_brute` | string |
| `statut` | string |
| `genre` | string |

## BD

| Colonne | Type |
|---|---|
| `id_unique` | string |
| `titre` | string |
| `categorie` | string |
| `albums_possedes` | number |
| `albums_lus` | number |
| `statut` | string |
| `albums_non_possedes_lu` | number |
| `albums_manquants` | number |
| `nb_dedicace` | string |

## Roman

| Colonne | Type |
|---|---|
| `id_unique` | string |
| `titre` | string |
| `auteur` | string |
| `genre` | string |
| `statut` | string |
| `nb_pages` | string |
| `Prix` | string |
| `editeur` | string |
| `annee_sortie` | number |

Documentation : [Data tables n8n](https://docs.n8n.io/data/data-tables/).
