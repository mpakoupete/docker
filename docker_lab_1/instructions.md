# LAB 1 - Découvrir Docker - Administrer Docker et Dockerfile

Cet TP sera effectué sur la machine hôte. Assurez vous d'avoir Docker installé.

## Installation de Docker

Se réferrer à [la documentation officielle](https://docs.docker.com/engine/install/ubuntu/)

<details><summary>Instructions d'installation</summary>

Mettre à jour l'index des paquets apt

```bash
sudo apt-get update
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Ajouter la clé GPG officielle de Docker

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Configurer le Dépot 

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Installation de Docker Engine

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

```

Vérifiez que l'installation de Docker Engine est réussie en exécutant l'image `hello-world`

```bash
docker version
sudo systemctl status docker
sudo docker run hello-world
```

</details>

## Explorez l'environnement Docker avec les commandes de base

À l'aide de quelques commandes de base de Docker, explorez l'environnement: 

* afficher l'aide de `docker` pour voir les options dont vous disposez

<details><summary>Correction</summary>

```bash
docker --help
```

</details>


* afficher la version de docker

<details><summary>Correction</summary>

```bash
docker version
```

</details>

* Lister les images présentes sur la machine

<details><summary>Correction</summary>

```bash
docker image ls
```

```bash
docker ps
```

</details>

* Lister les conteneurs en cours d'exécution sur la machine de deux manières

<details><summary>Correction</summary>

```bash
docker container ls
```

</details>

* Lancez un conteneur détaché du terminal dont l'image est `nginx:1.23`

<details><summary>Correction</summary>

```bash
sudo docker run -d nginx:1.23
```

</details>

* Lancez à nouveau un conteneur détaché du terminal dont l'image est `nginx:latest` et nom du conteneur `web` et inspecter ses caractérisques : container ID, adresse IP, Commande de lancement du conteneur ainsi que les arguments 

<details><summary>Correction</summary>

```bash
sudo docker run --name web -d nginx:latest
```

```bash
sudo docker inspect web
```

</details>

* visualisez les logs du conteneur `web`

<details><summary>Correction</summary>

```bash
sudo docker logs web
```

</details>

* Arreter le conteneur

<details><summary>Correction</summary>

```bash
sudo docker stop web
```

</details>

* Lister tous les conteneurs y compris ceux arrêtés sur la machine

<details><summary>Correction</summary>

```bash
sudo docker ps -a
```

</details>


## Ecrire Dockerfile

Créer un Simple Dockerfile qui crée une image à base de `nginx:1.25` avec les fichiers Web contenus dans le répertoire `"Ressources"` et les ports 80 et 443 devront être exposés.

<details><summary>Correction</summary>

Créer ce fichier Dockerfile dans le répertoire `"Ressources"`

```Dockerfile
FROM nginx:1.25

WORKDIR /usr/share/nginx/html
ADD . .

# Optionnel can déjà présent dans l'image nginx:1.25
EXPOSE 80 443 	

# Optionnel également can déjà présent dans l'image nginx:1.25
CMD ["nginx", "-g", "daemon off;"]
```

</details>

## Construire l'image de votre application - Build & Tag

Créer l'image (build) avec comme tag `<logindockerhub>/lab1-simpleapp:1.0`

<details><summary>Correction</summary>

```Bash
sudo docker build -t mpakoupete/lab1-simpleapp:1.0 .
```

</details>

Faites un push de l'image dans le regitry Docker

<details><summary>Correction</summary>

```Bash
sudo docker login
sudo docker push mpakoupete/lab1-simpleapp:1.0
```

</details>

## Lancer le conteneur

Lancer à présent le conteneur en mode détaché avec comme nom `simpleapp`

<details><summary>Correction</summary>

```Bash
sudo docker run --name simpleapp -d mpakoupete/lab1-simpleapp:1.0
```

</details>

Arrivez-vous à accéder aux site sur la machine host ?

<details><summary>Correction</summary>

Non. Car il faut faire un Port mapping pour accéder à l'intérieur du conteneur

</details>

Accéder à l'intérieur du conteneur et vérifier qu'avec le `curl localhost` le site est bien accessible.

<details><summary>Correction</summary>

```Bash
sudo docker ps
# identifiez l'ID de votre conteneur et accédez à l'intérieur du conteneur
sudo docker exec -it b0792f56c7e0 sh
```

</details>

## Lancer un autre conteneur

Lancez un autre conteneur `simpleapp2` avec le mapping de ports sur le 8080 de votre host
Vérifiez que le site web s'affiche bien `http://localhost:8080`

<details><summary>Correction</summary>

```Bash
sudo docker run --name simpleapp2 -p8080:80 -d mpakoupete/lab1-simpleapp:1.0
```

</details>

## Apportez une modification à l'image et mise à jour de l'image - Rebuild après modification

Faite une modification du fichier `index.html` et faites à nouveau un build de l'image avec pour tag :2
Lancez un autre conteneur `simpleapp2` à partir de cette nouvelle image avec le mapping de ports sur le 8081 de votre poste local.
Vérifiez que le site web s'affiche bien `http://localhost:8081`

<details><summary>Correction</summary>

```Bash
sudo docker build -t mpakoupete/lab1-simpleapp:2.0 .
sudo docker run --name simpleapp02 -p8081:80 -d mpakoupete/lab1-simpleapp:2.0
```

</details>

## Manipulation Docker

* Lister les images
* Lister les conteneurs
* Stoper le premier conteneur `simpleapp`
* Lister à nouveau les conteneurs en cours d'exécution et ensuite lister tous les conteneurs y compris ceux qui sont stoppés
* supprimer la première image de votre host
* Visualisez les logs du conteneur `simpleapp2` lorsque vous accédez au site web. Ensuite inspectez le.

<details><summary>Correction</summary>

```Bash
sudo docker images
sudo docker image ls
sudo docker container ls
sudo docker stop simpleapp
sudo docker container ls -a
sudo docker logs simpleapp2
sudo docker logs -f  simpleapp2
sudo docker container inspect simpleapp2
```

</details>
