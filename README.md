# BiblioBot

Collecte et statistiques de livres avec n8n : Bubble BD pour les BD, comics et mangas, Gleeph pour les romans, et un service Audible pour les livres audio. Un workflow Telegram et Ollama interroge les tables de livres.

## Sites concernés

| Livres | Site source | Collecteur |
|---|---|---|
| BD, comics et mangas | [Bubble BD — bubblebd.com](https://www.bubblebd.com/) | Mise à jour bibliothèque |
| Romans | [Gleeph — gleeph.com](https://www.gleeph.com/) | Mise à jour romans |
| Livres audio | [Audible France — audible.fr](https://www.audible.fr/) | Mise à jour Audible, via votre service personnalisé `audible-api` |

## Workflows

| Fichier | Contenu |
|---|---|
| [Mise à jour bibliothèque.json](WorkFlow/Mise%20%C3%A0%20jour%20biblioth%C3%A8que.json) | Bubble BD uniquement : collection, souhaits, PAL, statistiques et Google Sheets, 42 nœuds |
| [Mise à jour romans.json](WorkFlow/Mise%20%C3%A0%20jour%20romans.json) | Gleeph : romans, prix, pages, classements et Google Sheets, 27 nœuds |
| [Mise à jour Audible.json](WorkFlow/Mise%20%C3%A0%20jour%20Audible.json) | Livres audio, durées, prix estimé et Google Sheets, 22 nœuds |
| [Conseil Livre.json](WorkFlow/Conseil%20Livre.json) | Assistant tout-en-un Telegram/Ollama, 19 nœuds |

Le nom historique `Mise à jour bibliothèque.json` est conservé pour la branche Bubble BD. Les trois collecteurs sont indépendants : chacun possède le début planifié, la boucle de restauration et ses propres sorties Sheets.

## Nouveautés

- Ajout d’Audible : possession, souhaits, livres écoutés, durées totales et écoutées, minimum, maximum et moyenne. Le prix reste une estimation à 9,95 € par livre possédé, comme dans le code fourni.
- Bubble BD : pages, valeur et poids de collection, bulles lues, genres, auteurs, éditeurs et statistiques des séries.
- Gleeph : éditeur et année de sortie, répartitions par genres, auteurs, éditeurs et décennies, distinction possession/souhaits et extraction du prix global du site.
- Retrait des comptages spécifiques Batman, Spider-Man et Star Wars des trois nœuds de statistiques et de leurs mappings Sheets.
- Suppression de `Recherche de Tomes Manquants.json` et `Statistiques Globales.json` : leurs outils sont intégrés au workflow **Conseil Livre** fourni.

## Connexions propres aux services

**Bubble BD** : dans Configuration Globale, renseigner les quatre URL personnelles et `cookie_bubble`. Les cookies sont des tableaux JSON sérialisés ; `[]` est la valeur vide de publication. Configurer Browserless et le webhook Discord d’alerte.

**Gleeph** : renseigner l’URL `gleeph` et `cookie_gleeph`. La branche **HTTP Request3 → Code in JavaScript21** est conservée car **Gleephe1** dépend de son prix global. Le nœud Sheets partagé **BD3** n’écrit que les colonnes de sa branche ; il ne dépend plus d’un autre collecteur.

**Audible** : **HTTP Request Audible API** appelle `http://host.docker.internal:8010/audible/export`. Adapter cette URL à votre service. Le code de ce service personnalisé n’était pas fourni ; ce dépôt n’inclut donc pas son serveur ni ses fichiers d’authentification. Il doit renvoyer un objet `books` contenant des lignes conformes à `DataTables/Audible.csv`. Le nœud Code refuse un export sans livres. Les champs de cookies hérités du début ne configurent pas automatiquement l’authentification de ce service.

**Conseil Livre** : reconnecter Telegram et Ollama, sélectionner les tables BD et Roman dans le même projet et disposer du modèle `llama3.2:latest`, repris de l’export. Ce workflow fourni consulte actuellement BD et Roman ; le collecteur Audible est indépendant. L’assistant **Conseil Ultime** couvre aussi les livres audio. Le déclencheur de Conseil Livre reste Telegram, puisqu’une demande utilisateur est nécessaire.

## Docker

Le `docker-compose.yml` reprend les réglages de temps d’exécution de la configuration fournie, avec n8n, Browserless et Ollama. Pour une installation neuve, copier `.env.example` vers `.env`, adapter l’URL publique, puis lancer `docker compose up -d`. Avec une instance n8n existante, importer les workflows dans celle-ci. Configurer les credentials Ollama avec l’URL `http://ollama:11434` si ce service est utilisé.

Le service Audible est une dépendance externe à fournir séparément. Le volume n8n est créé par défaut ; pour réutiliser un volume existant, adapter sa déclaration avant le lancement.

## Démarrage et configuration

Chaque extraction planifiée reprend le début d’Ultime : **Schedule Trigger**, **Date & Time**, **Configuration Globale**, **Loop Over Items4**, **HTTP Request1**, **If5**. La branche d’échec conserve **Restauration Tunnel1**, **Wait1** et son retour dans la boucle.

1. Importer le JSON dans n8n. L’export reste désactivé tant que la configuration n’est pas terminée.
2. Remplacer l’URL `https://YOUR_N8N_HOST.example.invalid/` de **HTTP Request1** par celle de votre instance. Le succès est déterminé par un HTTP 200.
3. Reconnecter les credentials SSH du nœud de restauration et adapter la commande ngrok à votre installation. Sans tunnel, remplacer cette commande par votre propre mécanisme de restauration ou retirer explicitement cette branche.
4. Adapter l’horaire du planificateur : l’export reprend le vendredi à 17 h. Choisir le fuseau horaire de l’instance ou du workflow.
5. Remplacer les champs `YOUR_...`, renseigner les cookies privés lorsque nécessaires et sélectionner les credentials des services utilisés.
6. Créer les tables décrites dans [DataTables](DataTables/README.md), puis les sélectionner dans chaque nœud Data Table. Les CSV sont vides, avec leurs seuls en-têtes.
7. Pour les sorties Sheets, créer un onglet **N8N** à partir de [GoogleSheets/N8N.csv](GoogleSheets/N8N.csv), sélectionner votre document et vos credentials dans tous les nœuds Sheets. Les intitulés des colonnes et les colonnes de correspondance doivent rester identiques.
8. Vérifier une exécution complète avant activation. Les branches de collecte peuvent vider et reconstruire leurs tables cibles ; utiliser des tables dédiées.

Les identifiants de documents, tables, dossiers, comptes, cookies, tokens, données épinglées et historiques d’exécution personnels ont été retirés. Les identifiants internes des nœuds ont été régénérés. Les workflows séparés et le workflow complet sont des alternatives : éviter de lancer simultanément plusieurs versions qui reconstruisent les mêmes tables ou contrôlent le même tunnel.

## Vérification

Le JSON, les connexions, les références entre nœuds, la syntaxe JavaScript, les expressions complètes et les entrées des nœuds Merge ont été contrôlés localement. Aucun service personnel ni workflow de production n’a été exécuté. Les credentials et l’intégration réelle doivent être vérifiés dans l’instance cible.

Les six comparaisons de régression (tables complètes fournies et entrées vides pour BD, Roman et Audible) confirment que le retrait des compteurs par univers conserve les autres sorties des trois nœuds de statistiques.
