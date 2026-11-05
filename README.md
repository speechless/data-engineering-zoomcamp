# Commandes utiles
**Se connecter à la bdd postgre en cli**
uv run pgcli -h localhost -p 5432 -u root -d ny_taxi

**Lancer jupyter**
v run jupyter notebook

**Build le docker**
docker build -t test:pandas . 

**Lancer le docker**
docker run -it --rm -e POSTGRES_USER="root" -e POSTGRES_PASSWORD="root" -e POSTGRES_DB="ny_taxi" -v ny_taxi_postgres_data:/var/lib/postgresql -p 5432:5432 postgres:18


Avancée : 01-docker-terraform . 05-data-ingestion