# Kafka-mini-project
## contexe
Dans le cadre de ce projet on est sence  faire fonctionne le pipline de donnee, con stituees de ce composantes:
1. Kafka , kafka a comme role de capturer les donnee generee en sstreaming (simulation avec un script python `SendTempPressure.py` qui joue le role de producteur), les donnees sont des valeurs de tempertures et de pression, 
2. Telegraf , cette outil va nous permettre de configurer un consommateur Kafka qui va récupérer les données des topics states1 (températures) et states2 (pressions) depuis Kafka et ensuite les transférer vers comme InfluxDB.
5. InfluxDB est une base de données de séries chronologiques conçue pour stocker, interroger et visualiser des données temporelles. Une fois que Telegraf a récupéré les données depuis Kafka, il les stocke dans InfluxDB.
6. Grafana StackGrafana est un outil de visualisation de données open source. Vous pouvez l'utiliser pour créer des tableaux de bord interactifs et des graphiques basés sur les données stockées dans InfluxDB. ce qui nous va permettre de surveiller et d'analyser les données de température et de pression en temps réel.

## les problemes rencontrer
1. 
2. f
3. f
4. f
## le debougage et solutions:
