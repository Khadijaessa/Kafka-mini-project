# Kafka-mini-project
## contexe
Dans le cadre de ce projet, nous sommes censés construire un Pipeline de données avec Kafka et InfluxDB, avec les composants : 
1. Kafka , le rôle de kafka est de capturer les données générées en streaming (simulation avec un script python `SendTempPressure.py` qui joue le rôle de producteur), les données sont des valeurs de température et de pression.
2. Telegraf , cette outil va nous permettre de configurer un consommateur Kafka qui va récupérer les données des topics "states1" (températures) et "states2" (pressions) depuis Kafka et ensuite les transférer vers InfluxDB.
5. InfluxDB est une base de données de séries chronologiques conçue pour stocker, interroger et visualiser des données temporelles. Une fois que Telegraf a récupéré les données depuis Kafka, il les stocke dans InfluxDB.
6. Grafana Stack, Grafana est un outil de visualisation de données open source, on peut l'utiliser pour créer des tableaux de bord interactifs et des graphiques basés sur les données stockées dans InfluxDB. ce qui nous va permettre de surveiller et d'analyser les données de température et de pression en temps réel.

## Étapes du projet et débogage:

Clonez le dossier contenant les fichiers de configuration du pipeline de données:
```
 git clone https://github.com/hrhouma/Kafka-mini-projet-1.git
```
Utilisez cette commande pour démarrer les conteneurs Docker définis dans le fichier `docker-compose-test2.yml`:
 ```
 sudo docker compose -f docker-compose-test2.yml --env-file conf/variables.env up -d --pull always
```
 
Vérifier les conteneurs en cours d'exécution :
```
 docker ps
```

![data-pipeline (34)](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/72959eaa-b1a4-46c5-ba19-f7c2a3b44ffd)

Accéder aux interfaces des outils :

Utilisez les ports externes spécifiés dans la configuration de `docker-compose-test2.yml` pour accéder aux interfaces des outils :

Port 9000 pour KafDrop : http://localhost:9000

Port 8086 pour InfluxDB : http://localhost:8086

Port 3000 pour Grafana : http://localhost:3000

Se connecter à l'interface InfluxDB `(influx-admin:ThisIsNotThePasswordYouAreLookingFor)`, générer un nouveau token (lui donner tous les accès sur tous le buckets) et créer un bucket nommé "system_state" 

Ajouter le nouveau token d'accès et spécifier le nom du bucket ("system_state") dans le fichier d'environnement `variables.env`.

![variables](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/835c70d5-9324-47ba-afdb-c341817869a4)

Apres avoir configuerer l'environment pour kafka-python dans de dossier `DataSender`, on execute le script python avec la commande:

 ```
 python SendTempPressure.py
```

 ![image](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/4bd2e775-729a-40f6-bd3e-44b3743920d1)
 
### problème 1:
Les données ont été envoyées aux topics, on peut les voir via l'interface KafDorp:

![STATEES1](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/3424f358-14ca-4041-8a17-0526f6076ae5)

![STATEES2](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/6303f120-5e66-4c76-8540-561acff9ab8c)

Mais pas dans le bucket créé dans Influxdb.

![empltybucket](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/452bf9e9-1794-4a60-8ad4-fee7fe8b2605)

Ce problème peut être résolu en réfléchissant au fil manquant qui peut relier nos topics à notre base de données.

On ouvrant le fichier `telegraf.conf`, on voit deux parties de configuration.

La première correspond à l'output qui est la base de donnees Influxdb :
```
###############################################################################
#                            OUTPUT PLUGINS                                   #
###############################################################################

# Configuration for influxdb server to send metrics to
[[outputs.influxdb_v2]]
  ## The full HTTP or UDP endpoint URL for your InfluxDB instance.
  ## Multiple urls can be specified as part of the same cluster,
  ## this means that only ONE of the urls will be written to each interval.
  # urls = ["udp://localhost:8089"] # UDP endpoint example
  urls = ["http://influxdb:8086"] ## Docker-Compose internal address
  token = "random_token" ## token name, setting from config not working
  organization = "ORG" ## orga name, setting from config not working
  bucket = "system_state" ## bucket name / db name, setting from config not working
  #database = "system_state"

  ## Write timeout (for the InfluxDB client), formatted as a string.
  ## If not provided, will default to 5s. 0s means no timeout (not recommended).
  timeout = "1s"
 ```
La première remarque: 

- Il faut remplacer le token avec le token générer et configurer dans `variables.env`.

  > On peut aussi changer notre configuration de l'output , on mettant `token="$INFLUXDB_TOKEN"`
  > Et executer la commande :
  > ```
  > export INFLUX_TOKEN="MON_NOUVEAU_INFLUXDB_TOKEN"
  > ```
  > Avant d'exécuter le fichier `telegraf.conf`.

La deuxième partie de fichier correspond à l'input kafka consumer:

```
###############################################################################
#                            SERVICE INPUT PLUGINS                            #
###############################################################################

[[inputs.kafka_consumer]]
  ## Kafka brokers.
  brokers = ["kafka1:9092", "localhost:9093"] ## docker-compose internal address of kakfa
  
  ## Topics to consume.
  topics = [ "states"] ## topic to subscribe to

  ## When set this tag will be added to all metrics with the topic as the value.
  #topic_tag = "kafka"

  ## Optional Client id
  client_id = "kti_state" ## "username" of telegraf for kafka

  ## Set the minimal supported Kafka version.  Setting this enables the use of new
  ## Kafka features and APIs.  Must be 0.10.2.0 or greater.
  ##   ex: version = "1.1.0"
  # version = ""
```
La deuxième remarque:

- Dans cette partie de configuration il faut changer le topic "states" avec les topic "states1" et "states2".

![consumer](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/d60b0bef-a757-4265-826e-ca7415d30464)

 
Reconstruire les conteneurs:

 ```sudo docker compose -f docker-compose-test2.yml --env-file conf/variables.env up -d --pull always```

Après avoir exécuté la commande suivante dans le chemin où se trouve le fichier `telegraf.conf`, 

 ```
 telegraf --config telegraf.conf
```
 
 j'ai eu :
 
![config1](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/b726f02f-a618-4ef3-b7ae-822d09950595)

### Problème 2:

Des messages d'erreur commencent à apparaître :

![errormssg](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/38a9924e-d44d-4218-bef6-064a75e30331)

Cette erreur indique que le problème est dans l'IP du output qui est Influxdb, on explorant l'interface d'Influxdb, la partie telegraf nous pouvons voir une configuration du output:

![consumertelegr1](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/e00b64b2-7c61-40ac-ba10-d850914fa34e)

Dans cette configuration on vois qui'ils ont met comme adresse IP du Influx `http://localhost:8086` qui est l'adresse externe du Influxdb au lieu de l'adresse interne du conteneur `http://influxdb:8086`,

![data-pipeline (30)](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/e8e10c57-dde9-4488-a12f-dc870e9e154a)

J'ai configuré le fichier avec cette adresse,

![data-pipeline (35)](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/25f5b82a-6001-4e05-8bc2-3e2332fc0000)

Et après avoir reconstruit le pipeline et exécuté la configuration `telegraf.conf` et le script python, les données arrivent dans le bucket :

![reussi2](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/107d0b29-afb4-4e13-ba4d-8b2bab290f89)

L'erreur à été bien résolu.

> En résolvant ce problème de cette façon, j'ai pensé que nous pourrions modifier le docker-compose afin qu'il puisse se connecter directement à influxdb avec l'adresse interne `http://influxdb:8086`

Arrêter les conteneurs:

```
sudo docker compose -f docker-compose-test2.yml --env-file conf/variables.env down
``` 

Kafka ne reconnaît pas Influxdb comme output, Car ils n'ont pas de réseau commun j'ai donc ajouté le Network `db` à kafka dans Docker Compose :

![data-pipeline (32)](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/282e80a4-3035-4804-9a9e-73e3c8e5ad3d)

> On exécute la commande pour démarrer les conteneurs avec le nouveau fichier docker-compose, et on refait toutes les étapes.

Et ça a marché aussi, et j'avais les données dans mon bucket, avec une différence dans le nom d'hôte, puisque cette fois nous communiquons avec le conteneur `influxdb` avec son adresse interne.

![data-pipeline (33)](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/fdd26738-89f8-4c0e-8c16-1762a386ed70)

### Problème 3:

A ce stade, j'ai essayé de lire le bucket avec Grafana, en saisissant les informations sur la base de données Influxdb :

![grafanalocalhost](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/9720d8c2-0951-4dc9-92db-5f3b7bb01365)

J'ai eu l'erreur suivante:

![BUCKETERROR](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/ea8482ee-5b1b-4992-9263-20f245a16380)

Pour m'en assurer, j'ai essaie de me connecter à Influxdb dans le conteneur grafana, avec les commandes :

Pour accéder au conteneur:

```
docker exec -it grafana /bin/bash
```
Pour tester la connectioon:

```
curl  http://localhost:8086/ping
```
![data-pipeline (31)](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/eedc1dd7-56a9-40c8-b720-527739cfef4b)

Le problème était également dû à l'adresse utilisée, j'ai utilisé l'adresse `http://localhost:8086`, tandis que grafana communique avec Influxdb en interne et elle connu influxdb comme nom de service, on doit donc utiliser ` http : //influxdb:8086` dans Grafana.

![image](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/387d7015-61f3-402d-bb49-864d3981c59a)

Et finalement ouais, nous pouvons faire des rapports! 

![GAFANA](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/5e9c257a-78bb-449d-85d9-4a0d6a6763b8)

## conclusion:
 En faisant cette pratique, on peut mieux comprendre,
 - Les connexions internes et externes des conteneurs et comment gérer l'implémentation des réseaux dans docker-compose, pour connecter les différents composants.
 - L'utilisation des variables d'environnement dans Docker Compose pour spécifier les ports sur lequel un service doit écouter par exemple, ou pour stocker des informations sensibles telles que les identifiants et les mots de passe , et gérer les tokens et le nom des bucket comme dans le cas de Influxdb. 









