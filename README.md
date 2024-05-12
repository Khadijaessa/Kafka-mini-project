# Kafka-mini-project
## contexe
Dans le cadre de ce projet on est sence  faire fonctionne le pipline de donnee, con stituees de ce composantes:
1. Kafka , kafka a comme role de capturer les donnee generee en sstreaming (simulation avec un script python `SendTempPressure.py` qui joue le role de producteur), les donnees sont des valeurs de tempertures et de pression, 
2. Telegraf , cette outil va nous permettre de configurer un consommateur Kafka qui va récupérer les données des topics states1 (températures) et states2 (pressions) depuis Kafka et ensuite les transférer vers comme InfluxDB.
5. InfluxDB est une base de données de séries chronologiques conçue pour stocker, interroger et visualiser des données temporelles. Une fois que Telegraf a récupéré les données depuis Kafka, il les stocke dans InfluxDB.
6. Grafana StackGrafana est un outil de visualisation de données open source. Vous pouvez l'utiliser pour créer des tableaux de bord interactifs et des graphiques basés sur les données stockées dans InfluxDB. ce qui nous va permettre de surveiller et d'analyser les données de température et de pression en temps réel.

## etapes du projet et debougage:

Cloner le dossier et exécuter le docker-compose :

Clonez le dossier contenant les fichiers de configuration du pipeline de données:
```
git clone https://github.com/hrhouma/Kafka-mini-projet-1.git
```
Utilisez cette commande pour démarrer les conteneurs Docker définis dans votre fichier docker-compose.yml:
 ```sudo docker compose -f docker-compose-test2.yml --env-file conf/variables.env up -d --pull always```
 
Vérifier les conteneurs en cours d'exécution :
```
docker ps
```
S'ssurez que les conteneurs pour Kafka, InfluxDB et Grafana sont tous en cours d'exécution,
Accéder aux interfaces des outils :

Utilisez les ports spécifiés dans la configuration de `docker_compose.yaml` pour accéder aux interfaces des outils :
Port 9000 pour KafDrop : http://localhost:9000
Port 8086 pour InfluxDB : http://localhost:8086
Port 3000 pour Grafana : http://localhost:3000

Se connecter à l'interface InfluxDB via le port 8086, générez un token d'accès et créer un bucket nommé "system_state" .
Apres avoir configuerer l'environment pour kafka-python dans de dossier `DataSender`, on execute le script python avec la commande:
 ```python SendTempPressure.py```
### probleme 1:
les données ont été envoyées aux sujets, on peut les voir via l'interface KafDorp, mais pas dans le bucket créé en influx.

![image](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/4bd2e775-729a-40f6-bd3e-44b3743920d1)
![empltybucket](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/452bf9e9-1794-4a60-8ad4-fee7fe8b2605)

Ce problème peut être résolu en réfléchissant au fil manquant qui peut relier nos topics à notre base de données.
on ouvrant le fichier `telegraf.conf`, on voit deux parties de configuration.
le premier correspond au output qui est la base de donnees influx :
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

- Il faut remplacer le token avec le token generer et configurer dans`variables.env`.

La deuxième partie de fichier correspond a l'input kafka consumer:
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
- dans cette partie de configuration il faut changer le topic "states" avec les topic "states1" , "states2".
  
![topics](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/a5dd80d3-7035-490c-b671-8b68b0e7dc68)

Après avoir exécuté la commande dans le chemin où se trouve le fichier `telegraf.conf`, 

 ```telegraf --config telegraf.conf```
 j'ai eu :
 
![config1](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/b726f02f-a618-4ef3-b7ae-822d09950595)

## probleme 2
Lorsque j'exécute le script python, des messages d'erreur commencent à apparaître :

![errormssg](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/38a9924e-d44d-4218-bef6-064a75e30331)

cette erreur indique que le problème est dans l'IP du output qui Influx, on explorant l'interface d'Influxdb, la partie telegraf nous pouvons voir une configuration du output:

![consumertelegr1](https://github.com/Khadijaessa/Kafka-mini-project/assets/123899056/e00b64b2-7c61-40ac-ba10-d850914fa34e)

dan cette configuration on vois il ont met comme IP du Influx ``





