# Partie 1 : Docker - terraform
**Se connecter à la bdd postgre en cli**
``uv run pgcli -h localhost -p 5432 -u root -d ny_taxi``

**Lancer jupyter**
``uv run jupyter notebook``

**Lancer la DB Postgre et AdminPG**
(depuis 01-docker-terraform)
``docker compose up``

**Remplir la DB Postgre avec nos données**
``docker run -it --rm --network=data-engineering-zoomcamp_default taxi_ingest:v001 --pg-host=pgdatabase --pg-user=root --pg-pass=root --pg-port=5432 --pg-db=ny_taxi --target-table=yellow_taxi_trips``

les données de **zones**
``docker run -it --rm --network=data-engineering-zoomcamp_default taxi_ingest:v001 --pg-host=pgdatabase --pg-user=root --pg-pass=root --pg-port=5432 --pg-db=ny_taxi --target-table=zones --db=0 ``

# Partie 2 : Workflow orchestration
On lancer tout à partir du `docker compose up`

Le docker compose contient 4 instances :
- Kestra
- PG Admin
- Une DB Postgre contenant les données de Kestra
- Une DB Postgre pour les données produites par la pipeline

section 06: si on utilise des variables qui doivent charger des valeurs, une technique est de les appeler avant (comme dans un label) pour les charget et ne pas avoir la raw string

Kestra permet d'automatiser des tâches dont des manipulations de données, liens avec des DB, permet de les visualiser ou encore de les programmer.
section 08 : backfills permet de simuler une exécution dans un intervalle donné dans le passé.

section 9: ETL est un strandard dans l'industrie car ça fonctionne bien en général.
ELT permet d'utiliser la puissance de calcul du Cloud plutôt que celle en local. 
En pratique on envoie les données non structurées dans le DataLake (le Cloud dans notre cas) et on utilise ensuite BigQuery pour map ces données pour ensuite les structurer.
Cela permet d'avoir des grosses transformations plus rapidement car BigQuery ne charge que les données dont on a besoin, pas toutes (pas tout le fichier .csv dans notre cas).

section 10: Création des liens entre Kestra, BigQuery et GCP
Bien mettre les infos dans un .env et les importer depuis le docker-compose.yml
Pour les utiliser depuis Kestra, créer un fichier avec comme préfix SECRET_ devant chaque clé et encoder les valeurs en base64. Passer ce fichier dans le docker 
et il sera accessible depuis Kestra avec `secret("Nom_Du_Secret_Sans_Prefix")`

section 12: Avec le dataset des taxis, sur 3 mois, on arrive déjà à plus de 22M de lignes. D'où l'utilité d'utiliser parfois le cloud plutôt qu'un pauvre PC/serveur local à la longue.

section 16: on peut ajouter Gemini, faire des RAG simples, etc.

