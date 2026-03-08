# BiblioBot
# 📚 AI-Powered Personal Librarian (n8n + Ollama + Browserless)

Un système d'automatisation complet, souverain et 100 % hébergé en local, construit sur **n8n** pour extraire, synchroniser et interagir avec une vaste collection de Bandes Dessinées, Mangas, Comics (via *Bubble BD*) et Romans (via *Gleeph*). 

Ce projet inclut un pipeline ETL (Extraction, Transformation, Chargement) avancé contournant les limitations des sites web modernes, un système d'alerte de sécurité, et un **Agent IA local (Llama 3.1)** capable de répondre aux questions sur la collection en langage naturel.

La particularité de cette architecture est son optimisation : l'intégralité de la stack (scraping headless, bases de données, automatisation et LLM) est conçue pour tourner de manière fluide sur un **Raspberry Pi 5 (16 Go de RAM)** via Docker, garantissant une confidentialité totale (Self-Hosted) et l'absence de coûts d'API cloud.



## ✨ Fonctionnalités Principales

### 🕸️ 1. Web Scraping Avancé & Contournement Anti-Bot
* **Bubble BD :** Extraction automatique de la collection, des albums lus, des tomes manquants, et des statistiques globales (poids, valeur en €, pages lues).
* **Gleeph (Bypass du DOM Virtuel) :** Utilisation d'un script Puppeteer natif agressif ("La Moissonneuse") qui simule des coups de molette de souris physiques, auto-clique sur les boutons "Voir plus", et sauvegarde le code HTML à la volée pour contourner la limite d'affichage et le *Lazy Loading* sans perte de données.

### 🧠 2. Agent IA Local (Ollama)
Intégration d'un chatbot propulsé par des modèles LLM locaux via les nœuds *Advanced AI* de n8n. L'Agent agit comme un bibliothécaire personnel doté de "Tools" :
* **Data Store Tools :** Permet à l'IA de requêter dynamiquement la base de données pour vérifier si l'utilisateur possède une œuvre ou s'il lui manque des tomes.
* **Call Workflow Tools :** Décharge le LLM des opérations mathématiques (ex: "Combien ai-je de livres au total ?") via un workflow de pré-comptage, protégeant ainsi la fenêtre de contexte (*Context Window*).

### 🛡️ 3. Architecture Sécurisée & Centralisée (Best Practices n8n)
* **Configuration Globale :** Un nœud unique en début de workflow centralise les cookies de session et les URL Webhooks. Ces données sont injectées dynamiquement dans tous les nœuds HTTP via des expressions régulières (`{{ $json... }}`).
* **Cookie Guard :** Des mini-requêtes testent la validité des cookies *avant* de lancer les lourdes extractions.
* **Alertes Discord :** En cas d'expiration d'un cookie ou de déconnexion, le workflow s'interrompt proprement et alerte l'utilisateur via un Webhook Discord.

### 🧮 4. Algorithme de Déduction Logique
Création d'un script JavaScript personnalisé pour corriger les failles d'affichage des plateformes (ex: séries à la fois dans "Possédées" et "Souhaits"). Le script croise les données et calcule mathématiquement la différence exacte entre les tomes possédés et les tomes lus pour garantir des statistiques irréprochables, au tome près.

---

## 🛠️ Stack Technique

* **Matériel Cible :** Serveur Local / NAS / Raspberry Pi 5 (Architecture ARM64 ou AMD64)
* **Déploiement :** Docker & Docker Compose
* **Orchestrateur :** [n8n](https://n8n.io/) (v2.9.4 ou supérieur)
* **Scraping Headless :** Browserless (Chromium) via appels Puppeteer/Playwright
* **Intelligence Artificielle :** Ollama (LLM recommandé : `llama3.1:latest`)
* **Bases de données :** n8n Data Stores (Natif) & Google Sheets
* **Notifications :** API Discord (Webhooks)

---

## ⚙️ Architecture des Workflows n8n

Le dépôt contient les schémas JSON prêts à être importés dans n8n, divisés en deux workflows :

1. **`Mise à jour bibliothèque.json` (L'Usine à Données) :** Géré par un déclencheur planifié (Schedule Trigger), ce workflow charge les cookies, teste les accès, instancie Browserless pour aspirer Bubble BD et Gleeph, nettoie la donnée (Regex + JS), met à jour les Data Stores n8n (`Upsert`) et génère les statistiques.
   
2. **`Conseil Livre.json` (Le Cerveau IA) :** Un Chat Trigger connecté à un *AI Agent* avec mémoire tampon. L'Agent dispose d'outils natifs pour analyser en direct la base de données alimentée par le premier workflow.

---

## 🚀 Installation & Déploiement

### 1. Préparer l'environnement Docker
Clonez ce dépôt et lancez les conteneurs n8n et Browserless (déjà optimisés pour limiter la consommation de RAM) :
```bash
docker-compose up -d
```
2. Configurer Ollama (IA Locale)
Installez Ollama sur votre machine hôte (ou dans un autre conteneur) et téléchargez le modèle :

```bash
ollama run llama3.1:latest
```
(Note : Le docker-compose.yml inclut host.docker.internal pour que n8n puisse communiquer nativement avec l'Ollama de votre machine hôte).

3. Importer et Configurer les Workflows
Ouvrez votre instance n8n (http://localhost:5678).

Importez les deux fichiers .json.

Dans le workflow d'extraction, ouvrez le premier nœud "Configuration Globale" et collez :

Vos Cookies Bubble BD et Gleeph (Formatés en tableau JSON [...]).

Votre URL de Webhook Discord.

Activez les workflows !

Créé avec ❤️ pour les collectionneurs et les passionnés d'automatisation locale.


C'est parfait ! Ton projet est officiellement un bijou d'architecture et de code.
