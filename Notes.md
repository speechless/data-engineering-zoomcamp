# Partie 1 : Docker - terraform
**Se connecter à la bdd postgre en cli**
uv run pgcli -h localhost -p 5432 -u root -d ny_taxi

**Lancer jupyter**
uv run jupyter notebook

**Lancer la DB Postgre et AdminPG**
(depuis 01-docker-terraform)
docker compose up

**Remplir la DB Postgre avec nos données**
docker run -it --rm --network=data-engineering-zoomcamp_default taxi_ingest:v001 --pg-host=pgdatabase --pg-user=root --pg-pass=root --pg-port=5432 --pg-db=ny_taxi --target-table=yellow_taxi_trips

docker run -it --rm --network=data-engineering-zoomcamp_default taxi_ingest:v001 --pg-host=pgdatabase --pg-user=root --pg-pass=root --pg-port=5432 --pg-db=ny_taxi --target-table=zones --db=0 

# Partie 2 : Workflow orchestration

Le docker compose contient 4 instances :
- Kestra
- PG Admin
- Une DB Postgre contenant les données de Kestra
- Une DB Postgre pour les données produites par la pipeline
