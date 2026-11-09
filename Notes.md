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

**section 06**: si on utilise des variables qui doivent charger des valeurs, une technique est de les appeler avant (comme dans un label) pour les charget et ne pas avoir la raw string

Kestra permet d'automatiser des tâches dont des manipulations de données, liens avec des DB, permet de les visualiser ou encore de les programmer.

**section 08**: backfills permet de simuler une exécution dans un intervalle donné dans le passé.

**section 09**: ETL est un strandard dans l'industrie car ça fonctionne bien en général.

ELT permet d'utiliser la puissance de calcul du Cloud plutôt que celle en local. 

En pratique on envoie les données non structurées dans le DataLake (le Cloud dans notre cas) et on utilise ensuite BigQuery pour map ces données pour ensuite les structurer.

Cela permet d'avoir des grosses transformations plus rapidement car BigQuery ne charge que les données dont on a besoin, pas toutes (pas tout le fichier .csv dans notre cas).

**section 10**: Création des liens entre Kestra, BigQuery et GCP

Bien mettre les infos dans un .env et les importer depuis le docker-compose.yml

Pour les utiliser depuis Kestra, créer un fichier avec comme préfix SECRET_ devant chaque clé et encoder les valeurs en base64. Passer ce fichier dans le docker 
et il sera accessible depuis Kestra avec `secret("Nom_Du_Secret_Sans_Prefix")`

**section 12**: Avec le dataset des taxis, sur 3 mois, on arrive déjà à plus de 22M de lignes. D'où l'utilité d'utiliser parfois le cloud plutôt qu'un pauvre PC/serveur local à la longue.

**section 16**: on peut ajouter Gemini, faire des RAG simples, etc.

# Partie 3 - Data Warehouse & BigQuery
(pas de fichiers à mettre dans le repo, tout se passe sur BigQuery et GCP directement)

OLTP : Orienté transactions, BDD classiques. Utilisation souvent historique des données.
OLAP : BDD non normalisées et orientées pour l'analyse de donnée, souvent très lentes à update. Utilisation avec de larges datasets. OLAP se base souvent sur du OLTP pour se remplir.

Data warehouse contient en général des données brutes, des méta data mais aussi des données récapitulatives de différentes sources de données.

Option dans BigQuery pour activer ou désactiver l'utilisation du cache à considérer. (Actif par défaut).

Utilisation Flat rate ou On demand (à favoriser). En fonction de si on veut on ou un espace de données fixe ou en fonction de notre utilisation. Pas du tout le même prix.

BigQuery permet d'importer des fichiers de différents formats (comme .csv) et créé automatiquement la table avec les bons types de valeurs sans importer le schema.

## Partitions
On peut partitioner nos données (par date par exemple) pour réduire le temps de calcul des requêtes. Cela évite de process des données si on sait qu'elles ne sont pas pertinentes (l'exemple donne ~10 fois moins de données traitées). Le logo est pas le même entre une partition est une table sur le UI de BigQuery.

``... PARTITION BY <la donnée sur laquelle on veut trier> AS SELECT <... nos données>``

On peut voir les partitions des tables avec `.INFORMATION_SCHEMA.PARTITIONS`

On peut faire des _PARTITIONTIME pour fractionnée les partitions en fonction d'une durée.

Que ça soit avec des _PARTITIONTIME ou des données avec des Time unit, en général on partitionne par **jour**. 
Si on en a besoin et énormément de données, on peut partitionner par **heures**.

Si on a vraiment peu de données, on peut faire par **mois** ou par **années**.

Le nombre de partitions est limité à 4000.

Si le résultat des partitionnements donne des résultats < 1Gb ou trop de partitions ou que trop de modifications de partitions sont nécéssaires quand on les modifies (paritions pas optimisées), le Clustering peut être + intéressant.

## Clusters
`... CLUSTER BY <la donnée sur laquelle on veut trier> AS ...`

Comme un GROUP BY. Jusqu'à 4 colonnes.

L'ordre de la colonne est super important pour trier efficacement.

Sauve encore un peu de quantité de données utilisées. (utile si data > 1 GB).

Automatic reclustering est fait pour quand une donnée est ajoutée, elle s'adapte aux clusters existants.

## Bonnes pratiques
- Ne pas faire de `SELECT *`
- Réfléchir et quantifier les requêtes avant de les run.
- Utiliser des Cluster et des Partitions et filtrer dessus.
- Faire attention aux insertions automatiques.
- Stocker des données intermédiaires souvent utilisées dans des tables.


Pour gagner encore du temps :
- Dénormaliser les données (contrairement à d'habitudes) et mettre plus de données dans une même table.
- Filter first, JOIN after. Optimiser les JOIN
- `WITH` est une subquery, pas une clause. Faire attention car ça prend beaucoup plus de temps.
- Éviter le Javascript (UDF)
- `ORDER` doit être utilisé à la toute fin d'une requête.

## Structure de BigQuery
Les données sont stockées par colonne individuellement. (Column-oriented) Ce qui explique pourquoi les ``SELECT *`` sont si contraingnants.

La capacité de calcul et les données sont stockés dans 2 hardwares différents. 
Pour que cela n'affecte pas le temps de calcul final, le réseau qui communique entre les 2 à une bande passante d'environ 1TB/s.

La requête est ensuite envoyé à un serveur qui la sous-divise parmis les serveurs.

## Machine Learning dans BigQuery
Il est également possible de faire du ML dans BigQuery. Cela a l'air plutôt rudimentaire mais cela peut être tuner.

On peut utiliser pas mal de modèles très simples avec des presets tout faits ou plus complexes avec des paramètres avec lesquels on peut jouer.

Les avantages principaux de le faire depuis BigQuery sont :
- La puissance de calcul des infras Google
- Pas besoin d'exporter la donnée, le modèle s'intègre où elle est.

Après on peut exporter notre modèle évidement.

## Partie 4 : Analytics Engineering
Lien entre Data Engineer et Data Analyst.
Le but est de rendre disponible les données vraiment utiles pour les analyses.

dbt existe en 2 utilisations : dbt Core (une solution open-source, en cli) et dbt Cloud (en SaaS et payant)

### Structure d'un projet dbt
Voici une structure appréciée pour ordonner le dossier du projet dbt

#### dbt_project.yml
- Donne les valeurs par défaut à dbt
- Indispensable pour run les commandes dbt
- Doit avoir le même profile que dans .dbt/profiles.yml

#### macros
- Logique réutilisable comme des fonctions Python
- Encapsule des logiques au même endroits

#### seeds
- Dossier où upload des fichiers sources (csv ou autres) pour les ajouter à dbt plus tard
- De préférence à ne pas utiliser. Problème à fix à la source


#### tests
- Tests d'assertions en SQL
- Tests unitaires
- Si une commande SQL retounrne plus de 0 lignes, le build dbt fail

#### snapshot
- Sauvegarde (image, snapshot quoi) de tables à un moment donné
- Bien pour retracer l'historique de colonnes (qui s'overwrite elles-mêmes par exemple)

#### models

##### staging
- Sources (raw tables depuis la BDD)
- Résultats de filtres simples comme des rename de colonnes ou des changements de types

##### intermediate
- Tout ce qui n'est pas brut et que ne doit pas être exposé au public
- Nettoyages importants ou logique complexe

##### marts
- Prêt pour être utiliser par les utilisateurs, pour des dashboards
- Nettoyé et correctement modélisé

