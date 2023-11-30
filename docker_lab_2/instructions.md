# LAB 2 - Réseaux, volumes

## Réseau

Créer deux réseaux de type bridge de noms `mynet1`et `mynet2`

<details><summary>Correction</summary>

```bash
docker network create mynet1
```

```bash
docker network create mynet2
```

</details>

Lister les réseaux.
Inspectez les deux réseaux créés.
Quelles sont les plages d'adressage de ces réseaux et le Gatway?

<details><summary>Correction</summary>

```bash
docker network ls
```

```bash
docker network inspect mynet1
```
```json
[
    {
        "Name": "mynet1",
        "Id": "472bc9987fb922571b5d9405b7df9561e208a6bf874763ed64bf6d464979ee82",
        "Created": "2022-11-15T03:04:28.396283399Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

</details>

Lister les interfaces réseaux de votre machine hôte et répérez des deux réseaux créés

<details><summary>Correction</summary>

```bash
ip addr
ip link
```

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:a2:6b:fd brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:3d:30:e3:85 brd ff:ff:ff:ff:ff:ff
6: br-472bc9987fb9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:fa:d0:d0:72 brd ff:ff:ff:ff:ff:ff
7: br-030afe7f71d0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default
    link/ether 02:42:24:5a:f6:a8 brd ff:ff:ff:ff:ff:ff
```

les interfaces `br-472bc9987fb9` et `br-030afe7f71d0` sont nos nouvelles interfaces
</details>

Créez un conteneur du nom de `mywebapp` en mode détaché avec l'image précédemment créée dans le Lab 1 et attachez le conteneur au réseau `mynet1` faites un mapping du port 80 du conteneur sur le port `8084` du host

<details><summary>Correction</summary>

```Bash
docker run --name mywebapp  --network mynet1 -p 8084:80 -d logindockerhub/lab1-simpleapp:2.0
```

</details>

Inspectez le conteneur `mywebapp` et constatez bien qu'il est attaché à `mynet1`

<details><summary>Correction</summary>

```Bash
docker container inspect mywebapp
```

</details>


Créez 2 conteneurs avec l'image `curlimages/curl`
* Le premier conteneur du nom de `client1` connecté au réseau `mynet1`
* Le deuxièmen conteneur du nom de `client2` connecté au réseau `mynet2`
Vérifiez que à partir du `client1` vous pouvez faire un curl sur l'IP du conteneur `mywebapp` contrairement au curl depuis le conteneur `client2`


<details><summary>Correction</summary>

On peut passer directement en option à ce conteneur l'addresse IP pour lequel on veut faire un curl

```Bash
docker run --name client1 --network mynet1 curlimages/curl "172.18.0.2"

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
```

```Bash
docker run --name client2 --network mynet2 curlimages/curl "172.18.0.2"

(Pas de réponse)
```

</details>


## Volumes

Créer un Volume du nom `myvol`

<details><summary>Correction</summary>

```bash
docker volume create myvol
```

</details>

Lister les volumes.
Inspectez les volumes.
Quelles est le répertoire dans lequel les données seront persistées ?

<details><summary>Correction</summary>

```bash
docker volume ls
```

```bash
docker volume inspect myvol
```
```json
[
    {
        "CreatedAt": "2022-11-15T04:21:33Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvol/_data",
        "Name": "myvol",
        "Options": {},
        "Scope": "local"
    }
]
```

</details>

Créez un Conteneur avec les caractéristiques suivantes :
* Nom du conteneur `mysql`
* Image `mysql:8`
* En mode détaché
* Mapping de port `3306` du conteneur avec `3306` du host
* Attachez le volume précédemment créé `myvol` au répertoire (`/var/lib/mysql`) à l'intérieur du conteneur afin que MySQL y persiste la donées
* Importer une variable d'environnement `MYSQL_ROOT_PASSWORD` dont la valeur est `mysqlpassword`

<details><summary>Correction</summary>

```bash
docker run --name mysql -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=mysqlpassword \
    -v myvol:/var/lib/mysql \
    mysql:8
```

</details>

Visualisez les logs de ce conteneur

<details><summary>Correction</summary>

```bash
docker logs mysql
```

</details>

Accédez à la base de données depuis l'intérieur du conteneur mysql. Constatez la présence d'une variable d'environnement `MYSQL_ROOT_PASSWORD`

<details><summary>Correction</summary>

```bash
docker exec -it mysql sh
```

```bash
env

HOSTNAME=7c346f9e7429
MYSQL_ROOT_PASSWORD=mysqlpassword
PWD=/
HOME=/root
MYSQL_MAJOR=8.0
GOSU_VERSION=1.14
MYSQL_VERSION=8.0.31-1.el8
TERM=xterm
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MYSQL_SHELL_VERSION=8.0.31-1.el8
_=/usr/bin/env
```

```bash
mysql -u root -h localhost -p
```

</details>

Créer une base de données et une tables puis insérez des données.

<details><summary>Correction</summary>

```bash
create database etudiants;
```

```bash
use etudiants;
```

```bash
CREATE TABLE session_k8s (
    ID int,
    Nom varchar(255),
    Prenom varchar(255),
    Adresse varchar(255),
    Ville varchar(255)
);
```

```bash
INSERT INTO session_k8s (ID, Nom, Prenom, Adresse, Ville) VALUES (1, 'Test', 'Test', '1 rue albert,75001', 'Paris');
```

</details>

Quitter le conteneur et vérifier que les données se trouve dans le répertoire de persistance ce la donnée 

<details><summary>Correction</summary>

Sur la machine host

```bash
sudo ls /var/lib/docker/volumes/myvol/_data -l
total 99680
-rw-r----- 1 systemd-coredump systemd-coredump   196608 Nov 15 04:59 '#ib_16384_0.dblwr'
-rw-r----- 1 systemd-coredump systemd-coredump  8585216 Nov 15 04:36 '#ib_16384_1.dblwr'
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:37 '#innodb_redo'
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:37 '#innodb_temp'
-rw-r----- 1 systemd-coredump systemd-coredump       56 Nov 15 04:36  auto.cnf
-rw-r----- 1 systemd-coredump systemd-coredump  3023345 Nov 15 04:36  binlog.000001
-rw-r----- 1 systemd-coredump systemd-coredump     2156 Nov 15 04:59  binlog.000002
-rw-r----- 1 systemd-coredump systemd-coredump       32 Nov 15 04:37  binlog.index
-rw------- 1 systemd-coredump systemd-coredump     1680 Nov 15 04:36  ca-key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Nov 15 04:36  ca.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Nov 15 04:36  client-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1676 Nov 15 04:36  client-key.pem
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:59  etudiants
-rw-r----- 1 systemd-coredump systemd-coredump     5690 Nov 15 04:36  ib_buffer_pool
-rw-r----- 1 systemd-coredump systemd-coredump 12582912 Nov 15 04:59  ibdata1
-rw-r----- 1 systemd-coredump systemd-coredump 12582912 Nov 15 04:37  ibtmp1
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:36  mysql
-rw-r----- 1 systemd-coredump systemd-coredump 31457280 Nov 15 04:59  mysql.ibd
lrwxrwxrwx 1 systemd-coredump systemd-coredump       27 Nov 15 04:36  mysql.sock -> /var/run/mysqld/mysqld.sock
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:36  performance_schema
-rw------- 1 systemd-coredump systemd-coredump     1676 Nov 15 04:36  private_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump      452 Nov 15 04:36  public_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Nov 15 04:36  server-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1676 Nov 15 04:36  server-key.pem
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:36  sys
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:54  test
-rw-r----- 1 systemd-coredump systemd-coredump 16777216 Nov 15 04:59  undo_001
-rw-r----- 1 systemd-coredump systemd-coredump 16777216 Nov 15 04:59  undo_002
```

```bash
sudo ls /var/lib/docker/volumes/myvol/_data/etudiants -l
total 112
-rw-r----- 1 systemd-coredump systemd-coredump 114688 Nov 15 04:59 session_k8s.ibd
```

</details>

Supprimez le conteneur `mysql`

<details><summary>Correction</summary>

```bash
docker stop mysql
```

```bash
sudo docker rm mysql
```

</details>

Constatez que les données sont toujours existante dans le répertoire de persistance

<details><summary>Correction</summary>

Sur la machine host

```bash
sudo ls /var/lib/docker/volumes/myvol/_data -l
total 99680
-rw-r----- 1 systemd-coredump systemd-coredump   196608 Nov 15 04:59 '#ib_16384_0.dblwr'
-rw-r----- 1 systemd-coredump systemd-coredump  8585216 Nov 15 04:36 '#ib_16384_1.dblwr'
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:37 '#innodb_redo'
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:37 '#innodb_temp'
-rw-r----- 1 systemd-coredump systemd-coredump       56 Nov 15 04:36  auto.cnf
-rw-r----- 1 systemd-coredump systemd-coredump  3023345 Nov 15 04:36  binlog.000001
-rw-r----- 1 systemd-coredump systemd-coredump     2156 Nov 15 04:59  binlog.000002
-rw-r----- 1 systemd-coredump systemd-coredump       32 Nov 15 04:37  binlog.index
-rw------- 1 systemd-coredump systemd-coredump     1680 Nov 15 04:36  ca-key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Nov 15 04:36  ca.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Nov 15 04:36  client-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1676 Nov 15 04:36  client-key.pem
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:59  etudiants
-rw-r----- 1 systemd-coredump systemd-coredump     5690 Nov 15 04:36  ib_buffer_pool
-rw-r----- 1 systemd-coredump systemd-coredump 12582912 Nov 15 04:59  ibdata1
-rw-r----- 1 systemd-coredump systemd-coredump 12582912 Nov 15 04:37  ibtmp1
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:36  mysql
-rw-r----- 1 systemd-coredump systemd-coredump 31457280 Nov 15 04:59  mysql.ibd
lrwxrwxrwx 1 systemd-coredump systemd-coredump       27 Nov 15 04:36  mysql.sock -> /var/run/mysqld/mysqld.sock
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:36  performance_schema
-rw------- 1 systemd-coredump systemd-coredump     1676 Nov 15 04:36  private_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump      452 Nov 15 04:36  public_key.pem
-rw-r--r-- 1 systemd-coredump systemd-coredump     1112 Nov 15 04:36  server-cert.pem
-rw------- 1 systemd-coredump systemd-coredump     1676 Nov 15 04:36  server-key.pem
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:36  sys
drwxr-x--- 2 systemd-coredump systemd-coredump     4096 Nov 15 04:54  test
-rw-r----- 1 systemd-coredump systemd-coredump 16777216 Nov 15 04:59  undo_001
-rw-r----- 1 systemd-coredump systemd-coredump 16777216 Nov 15 04:59  undo_002
```

```bash
sudo ls /var/lib/docker/volumes/myvol/_data/etudiants -l
total 112
-rw-r----- 1 systemd-coredump systemd-coredump 114688 Nov 15 04:59 session_k8s.ibd
```

</details>


Créez un nouveau Conteneur avec les caractéristiques suivantes :
* Nom du conteneur `mysql2`
* Image `mysql:8`
* En mode détaché
* Attachez le même volume précédemment créé `myvol` au répertoire (`/var/lib/mysql`) à l'intérieur du conteneur
* Importer une variable d'environnement `MYSQL_ROOT_PASSWORD` dont la valeur est `rootroot01`

<details><summary>Correction</summary>

```bash
docker run --name mysql2 -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=rootroot01 \
    -v myvol:/var/lib/mysql \
    mysql:8
```

</details>

Accédez à la base de données depuis l'intérieur du conteneur mysql. Quel mot de passe avez vous utilisez ?

<details><summary>Correction</summary>

```bash
docker exec -it mysql2 sh
mysql -u root -h localhost -p
```

On a dû utilisé l'ancien mot de passe de la base de donnée précédente `mysqlpassword`

</details>

Lister les base de données et vérifiez que nos données précédemment créées sont présentes

<details><summary>Correction</summary>

```bash
show databases;
```

```bash
use etudiants;
```

```bash
show tables;
```

```bash
mysql> select * from session_k8s;

+------+------+--------+--------------------+-------+
| ID   | Nom  | Prenom | Adresse            | Ville |
+------+------+--------+--------------------+-------+
|    1 | Test | Test   | 1 rue albert,75001 | Paris |
+------+------+--------+--------------------+-------+
1 row in set (0.00 sec)
```

On voit bien la présence de nos anciennes données

</details>
