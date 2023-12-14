# Projet Docker

## Projet Docker 1 - Stack Applicative WordPress et MariaDB

**Objectif :** Mettre en place une stack applicative composée d'une application frontend WordPress et d'une base de données MariaDB, utilisant Docker et Docker Compose.

**Instructions :**

1. **Application WordPress :**
   - Choisissez une image de base de **système d'exploitation** optimisée pour WordPress.
   - Fournissez un fichier `Dockerfile` pour construire l'image. Justifiez votre choix d'image de base.
   - Assurez-vous que l'image contient les variables d'environnement nécessaires pour se connecter à une base de données MySQL/MariaDB.
   - Cette image doit être placé dans votre compte Docker Hub afin que sur ma machine, je puisse télécharger votre fichier `docker-compose.yml` et pouvoir exécuter la stack sans difficulté.

2. **Base de Données MariaDB :**
   - Choisissez une image de base de données MariaDB dans le registre Docker.
   - Fournissez un fichier `Dockerfile` si nécessaire et justifiez votre choix d'image.

3. **La Stack :**
   - Fournissez un fichier `docker-compose.yml` pour le lancement de la stack.
   - Configurez la base de données pour persister ses données même après l'arrêt et la suppression du conteneur.
   - Pour faciliter la configuration de la base de données MySQL, mappez un fichier `my.cnf` contenu dans le même répertoire où se trouve le fichier `docker-compose.yml`.

4. **Réseaux Virtuels Distincts :**
   - Créez deux réseaux virtuels distincts pour cette stack.
   - Le premier réseau (`frontend_network`) est utilisé par le frontend uniquement.
   - Le deuxième réseau (`backend_network`) est utilisé par les deux, la base de données et le frontend.

**Remarques :**
- Justifiez toutes vos décisions, y compris le choix des images de base, la configuration des variables d'environnement, et le choix des réseaux virtuels.
- Assurez-vous de respecter les meilleures pratiques en termes de sécurité et d'efficacité pour la configuration de la base de données et du frontend.
- Testez la stack pour vous assurer que WordPress peut se connecter à la base de données et que les données de la base de données persistent correctement même après le redémarrage de la stack.


## Projet Docker 2 - Cluster Elasticsearch avec Kibana

**Objectif :** Mettre en place un cluster Elasticsearch composé de trois nœuds et un conteneur Kibana qui a accès au cluster.

**Instructions :**

1. **Structure du Projet :**
   - Créez un répertoire nommé `votre-nom-de-binome`.
   - À l'intérieur de ce répertoire, créez une structure de fichiers et de répertoires selon le schéma suivant :

   ```
   votre-nom-de-binome/
   │
   ├── docker-compose.yml
   ├── .env
   ├── es-cluster/
           └── elasticsearch.yml
           └── kibana.yml
   ```

2. **Fichier .env :**
   - Créez un fichier `.env` à la racine du projet (`votre-nom-de-binome`).
   - Définissez une variable d'environnement `ES_VERSION` avec la valeur `8.11.3` pour faire référence à la version Elasticsearch et kibana qui s'exécuteront. chaque fois que cette variable est modifiée, le nouveau cluster déployé devrait correspondre à la version mqui y est entionnée.

3. **Configuration du cluster Elasticsearch :**
   - Dans le répertoire `es-cluster`, créez un fichier `elasticsearch.yml` avec la configuration Elasticsearch.
   - Dans ce même répertoire `es-cluster`, créez un fichier `kibana.yml` avec la configuration Kibana.

4. **docker-compose.yml :**
   - Créez un fichier `docker-compose.yml` à la racine du projet avec la configuration suivante :
     - Trois services Elasticsearch (`es01`, `es02`, `es03`) qui forment un cluster.
     - Un service Kibana (`kibana`) connecté au cluster Elasticsearch.
   - Assurez-vous que les nœuds Elasticsearch utilisent les fichiers de configuration appropriés.
   - Configurer les variables d'environment nécessaire.

5. **Exécution :**
   - Ouvrez un terminal, placez-vous dans le répertoire `votre-nom-de-binome`, et exécutez la commande `docker-compose up`.
   - Vérifiez que le cluster Elasticsearch et le service Kibana sont opérationnels.

**Remarques :**
- La configuration du cluster Elasticsearch doit spécifier trois nœuds (`es01`, `es02`, `es03`).
- Les données doivent être persistents
- Le service Kibana doit être configuré pour se connecter à l'un des nœuds Elasticsearch.
- Configurez un réseau sur lequel sont connectés tous les services.

## Rendu du projet

Vous devez rendre :
- Un fichier docs/word avec les explications et captures d'écran nécessaire
- Le code (fichier compose, Dockerfile...) bien commenté pour une meilleur compréhension
- Vous pouvez juste m'envoyer un lien GitHub où tout cela se trouve.