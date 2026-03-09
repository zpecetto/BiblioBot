# 📚 BiblioBot : Le Bibliothécaire IA Auto-Hébergé
BiblioBot est un système d'automatisation complet, souverain et 100% auto-hébergé. Il permet d'extraire, synchroniser et interagir en langage naturel avec vos collections de Bandes Dessinées, Mangas, Comics (via Bubble BD) et Romans (via Gleeph).

L'architecture est optimisée pour tourner de manière fluide sur un Raspberry Pi 5 (16 Go RAM), garantissant une confidentialité totale sans aucun coût d'API cloud.

🚀 Points Forts
Pipeline ETL Avancé : Extraction et transformation de données complexes depuis des plateformes sans API officielle.

IA Locale (RAG) : Un agent Qwen 2.5 (ou Llama 3.1) capable de répondre à des questions précises sur votre collection (ex: "Me manque-t-il des tomes de Batman ?").

Anti-Bot & Lazy Loading : Utilisation de Browserless pour simuler une navigation humaine et capturer le DOM dynamique (Bypass du Virtual DOM).

Souveraineté Totale : Aucune donnée ne quitte votre réseau local (hormis la synchronisation optionnelle vers Google Sheets).

✨ Fonctionnalités détaillées
🕸️ 1. Scraping & Moissonneur HTML

Bubble BD : Récupération automatique des albums possédés, lus, manquants et statistiques (valeur €, poids, nombre de pages).

Gleeph (La Moissonneuse) : Script Puppeteer agressif simulant des interactions physiques (scroll, clics "Voir plus") pour aspirer l'intégralité des catalogues malgré le Lazy Loading.

🧠 2. Agent IA Local (Ollama)
L'agent utilise les nœuds Advanced AI de n8n pour agir comme un vrai bibliothécaire :

Outils de Données : Requête dynamiquement les Data Tables n8n pour vérifier vos stocks.

Opérations Mathématiques : Les calculs lourds (comptage total) sont délégués à des sous-workflows pour économiser la fenêtre de contexte du LLM et éviter les hallucinations.

🛡️ 3. Architecture & Sécurité

Configuration Centralisée : Un nœud unique gère les cookies de session et Webhooks pour faciliter la maintenance.

Cookie Guard : Système d'alerte via Discord en cas d'expiration de session ou d'échec d'extraction.

Algorithme de Déduction : Script JS personnalisé pour corriger les erreurs de statut (ex: doublons entre listes "Souhaits" et "Possédés").

🛠️ Stack Technique
Serveur : Docker / Docker Compose (ARM64/AMD64).

Orchestration : n8n (v2.9.4+).

Navigateur Headless : Browserless (Chromium).

Modèle LLM : Ollama (qwen2.5:7b recommandé pour les appels d'outils, ou llama3.1:latest).

Stockage : n8n Data Tables & Google Sheets.

⚙️ Installation & Déploiement
1. Lancer l'infrastructure
Clonez le dépôt et lancez les conteneurs :

```Bash
docker compose up -d
```
2. Configurer l'IA (Ollama)
Assurez-vous qu'Ollama est installé sur l'hôte et téléchargez le modèle :

```Bash
ollama run qwen2.5:7b
```
Note : Le fichier docker-compose.yml utilise host.docker.internal pour lier n8n à Ollama.

3. Importer les Workflows (Ordre strict)
Ouvrez n8n (http://localhost:5678).

Importez Mise à jour bibliothèque.json (L'Usine à Données).

Importez Recherche de Tomes Manquants.json (Outil IA n°1).

Importez Statistiques Globales.json (Outil IA n°2).

Importez Conseil Livre.json (Le Cerveau IA).

⚠️ Étape Cruciale : Reconnecter les outils IA
Dans le workflow Conseil Livre, double-cliquez sur les deux nœuds "Call Workflow" et re-sélectionnez manuellement les sous-workflows Recherche de Tomes Manquants et Statistiques Globales dans la liste déroulante pour restaurer les ID.
Pour l'outil de Recherche : Assurez-vous que le paramètre d'entrée est défini sur Auto-Map Input Data (ou mappé manuellement avec ={{ $json.titre }}).

4. Configuration des identifiants

Cookies de session : Dans le nœud Configuration Globale du workflow principal, renseignez vos cookies Bubble/Gleeph sous forme de texte brut (String), copiés depuis votre navigateur.

Discord (Optionnel) : Renseignez votre Webhook Discord pour les alertes.

Ngrok (Optionnel) : Le workflow inclut des nœuds SSH pour gérer un tunnel ngrok. Si vous n'exposez pas votre instance, vous pouvez supprimer ces nœuds.

5. Liaison Google Sheets (Optionnel)
Pour chaque nœud Google Sheets présent dans le workflow :

Créez vos identifiants OAuth2 dans n8n.

Remplacez VOTRE_ID_GOOGLE_SHEET_ICI par l'ID réel de votre tableur.

Sélectionnez les onglets correspondants (BD, Manga, etc.).

Créé avec ❤️ pour les collectionneurs et les passionnés d'auto-hébergement.
