# Lab 3 - Docker compose

Nous nous mettons dans le contexte où nous avons développé une application `example-voting-app`
Pour rappel, [example-voting-app](https://github.com/dockersamples/example-voting-app) est une application maintenu par la communauté Docker qui est très souvent utilisée pour faire des démo de microservices.
Il est concu en microservices avec l'architecture suivante :

![architecture](https://user-images.githubusercontent.com/59444079/202094328-30a5cb07-f33b-40bf-828a-7dc5de405bb7.png)

La stack est composée de 5 services:
* un service frontend `vote` développé en Python : Les utilisateurs peuvent y accéder pour voter en cliquant
* un service frontend `result` développé en Node.js : Les utilisateurs peuvent y accéder pour voir les résultats du vote
* un service backend `worker` qui traites les requètes et les persistent dans une base de donnée PostgreSQL
* un service de Base de données `db` PostgreSQL qui persiste la donnée
* un service de cache `redis` utilisé par le microservice `voting-app`

L'objectif du Lab sera de Dockeriser cette stack et la lancer avec docker-compose, idéal pour un environment de Dev

## Partie 1 : Mettre en place la base données PostgreSQL


En utilisant le client `docker`, Lancer le conteneur du nom de `docker_db` :
* En mode détaché 
* Avec comme image `postgres:11.6-alpine`, 
* Le mapping des ports `5432` et du volume `db_data` que vous devez créer et attacher au répertoire `/var/lib/pgsql/data` à l'intérieur du conteneur
* importer les variables d'environement `"POSTGRES_USER=postgres"` et `"POSTGRES_PASSWORD=postgres"`

Accedez à l'intérieur du conteneur et vérifier que postgres est bien en cours d'execution.

<details><summary>Correction</summary>

```Bash
docker volume create db_data
docker volume ls
docker run --name docker_db --env="POSTGRES_USER=postgres" --env="POSTGRES_PASSWORD=postgres" -p:5432:5432 -d -v db_data:/var/lib/pgsql/data postgres:11.6-alpine
```

</details>

## Partie 2 : Le Front End

Télécharger le code source de la stack

Dans le répertoire `result` il y'a le code source d'une appli web AngulaJS qui affiche les résultats

Pour lancer l'application voici les commandes à exécuter dans le répertoire **result-app**

```bash
$ npm install
$ npm start
```

Si `npm` n'est pas installé sur votre machine vous devez l'installer : ' `apt-get install -y npm`

Constatez que l'application est lancé sur le port `4000`

Ecrire un Dockerfile pour dockeriser cette application result-app

Faite un Build avec comme tag `votrelogingdocker/result-app:latest`

Lancez un conteneur du nom `result-app` à base de cette image

<details><summary>Correction</summary>

Pour que le FT puisse démarrer qu'est ce que nous avons du faire ?

- npm install
- npm start

==> Donc la Solution pour dockeriser: Je dois installer dans un environnement où ces commanandes sont possibles
Il faudra que la commande npm existe

Npm est le gestionnaire de paquets de NodeJS ==> Solution. De base notre environnement sera donc une image NodeJS avec la version de mon environnement de test 
node version

==> on pourra pare par exemple 
    FROM node:16

Ensuite les tâches à faire :
    - Copier mes fichier source
    - Faire l'installation des dépendences
    - Démarrer notre application sur le port précis 4000

Exemple de Dockerfile pour un environnement de test:

```Dockerfile
FROM node:16

WORKDIR /app

COPY package*.json ./

RUN npm install 

COPY . .

EXPOSE 4000

CMD ["npm", "start"]
```


==> Voir le Dockerfile dans le répertoire `solution` qui a été fourni par Docker

Les commandes à saisir pour dockeriser l'application :

```bash
docker build -t mpakoupete/result-app:latest .

[+] Building 25.3s (11/11) FINISHED                                                                                                                                                                                                                   
 => [internal] load build definition from Dockerfile                                                                                                                                                                                             0.0s
 => => transferring dockerfile: 153B                                                                                                                                                                                                             0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                                0.0s
 => => transferring context: 54B                                                                                                                                                                                                                 0.0s
 => [internal] load metadata for docker.io/library/node:16                                                                                                                                                                                       1.6s
 => [auth] library/node:pull token for registry-1.docker.io                                                                                                                                                                                      0.0s
 => [1/5] FROM docker.io/library/node:16@sha256:31667a02978fc47061eaae324e161c58e143a3e0ed27474bf54a7a18b89509d7                                                                                                                                16.7s
 => => resolve docker.io/library/node:16@sha256:31667a02978fc47061eaae324e161c58e143a3e0ed27474bf54a7a18b89509d7                                                                                                                                 0.0s
 => => sha256:31667a02978fc47061eaae324e161c58e143a3e0ed27474bf54a7a18b89509d7 776B / 776B                                                                                                                                                       0.0s
 (..)                                                                                                                                                       0.1s
 => => extracting sha256:f0dfa5d235bc6119baa6a623d47ac9e98dd967a914d22a3d2f7ebec03c601a8c                                                                                                                                                        0.0s
 => [internal] load build context                                                                                                                                                                                                                0.0s
 => => transferring context: 252.77kB                                                                                                                                                                                                            0.0s
 => [2/5] WORKDIR /app                                                                                                                                                                                                                           1.8s
 => [3/5] COPY package*.json ./                                                                                                                                                                                                                  0.0s
 => [4/5] RUN npm install                                                                                                                                                                                                                        4.7s
 => [5/5] COPY . .                                                                                                                                                                                                                               0.0s
 => exporting to image                                                                                                                                                                                                                           0.3s
 => => exporting layers                                                                                                                                                                                                                          0.3s
 => => writing image sha256:1480489c261ed92aa37e728aaff8e254f9d5ef24bdf9c8fd8477cb4d59f85547                                                                                                                                                     0.0s
 => => naming to docker.io/mpakoupete/result-app:latest 

```

</details>

Testez en exécutant un conteneur avec mapping de port 4000 <=> 4000, vérifiez qu'on a bien accès à l'application `result-app`

<details><summary>Correction</summary>

```bash
docker run --name result-app -p4000:4000 -d mpakoupete/result-app:latest

# Vous devez voir Error connecting to db, c'est parce qu'il n'arrive pas à joindre la base de donnée
# La DB est exposée sur le port 5432 de la machine hôte. Dans le fichier /etc/hosts, on ajoutera la ligne suivante:
# <IP de l'interface eth0>  db

echo $(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}') db | sudo tee -a /etc/hosts

# après l'ajout de cette ligne, il devait pouvoir joindre la DB sauf que la base de donnée vote n'est pas encore créée. On peut s'en passer pour l'instant
```

</details>

## Partie 3 : Docker Compose file

Rédiger un `docker-compose.yml` qui mettra cette stack en place par une seule commande `$ docker-compose up`

<details><summary>Correction</summary>

Voir le fichier `docker-compose.yml` dans le répertoire de solution

```bash
# Lancer la stack en mode détaché
docker compose up -d 

# Lister le nombre de stack compose lancé
docker compose ls

# voir les contenaurs lancés 
docker compose ps

NAME                COMMAND                  SERVICE             STATUS              PORTS
solution-db-1       "docker-entrypoint.s…"   db                  running (healthy)   5432/tcp
solution-redis-1    "docker-entrypoint.s…"   redis               running (healthy)   0.0.0.0:50816->6379/tcp
solution-result-1   "docker-entrypoint.s…"   result              running             0.0.0.0:5001->80/tcp, 0.0.0.0:5858->5858/tcp
solution-vote-1     "python app.py"          vote                running             0.0.0.0:4000->80/tcp
solution-worker-1   "dotnet Worker.dll"      worker              running             

# voir les logs des service ==> on specifie le nom des service frontend
docker compose logs vote
docker compose logs result
docker compose logs worker
docker compose logs redis
docker compose logs db

# Arrêter tous les conteneurs
docker compose down

```

**Diagnostic réseau**

A partir d'un conteneur qui sera créé dans le même réseau, faire un test de connextion : Ping ou Curl

```bash
docker run --network <Nom du Réseau> --rm curlimages/curl curl http://172.22.0.2:3080
```

</details>
