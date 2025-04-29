# Notes for GSRS Docker embedded deployment

## Terminal Environment

```
export DOCKER_SOURCE=../../docker-source
export HOST_VOLUMES=../volumes

export DB_TEST_USERNAME=root
export DB_TEST_PASSWORD=yourpassword

# export RELEASE_MODE=development
export RELEASE_MODE=public

export BUILD_VERSION=v2024.1115.1

```

## Purpose

This Docker recipe is mainly meant for local testing and also to provide an introduction to using Docker with GSRS. 


## gsrs-ci

Below gsrs-ci refers to the deployments folder used by FDA. but you may use any similar deployment repository such as gsrs-example-deployment, or gsrs3-main-deployment.  

## Clone gsrs-ci

```
# Temporarily clone your gsrs-ci repo here
cd gsrs3-docker-deployments/projects
```

## Running the containers

First you'll need to build your images (see below)

```
# use ONE of (postgresql, mariadb, mysql) database flavors.
# The docker-compose.yml file should require one of these but does not yet do so.
# If you need to use sudo, put the sudo before the db credentials.

cd gsrs-ci

export DATABASE=postgresql 
sudo \
DB_TEST_USERNAME=root DB_TEST_PASSWORD=yourpassword \
docker-compose -f ../docker-source/docker-compose.yml up \
$DATABASE frontend gateway substances products
```

## Available services

```
adverse-events  ?
applications
discovery
frontend
gateway 
clinical-trials
impurities
invitro-pharmacology
substances
ssg4m

# Most entity services, depend on the substances service for full fuctionality. 
# Discovery isn't used in practice, yet.
```

## Pick one database service
```
h2 
mariadb 
mysql 
postgresql

#  For h2, set $DATABASE='' on the docker-compose command. It is used by default
```

## Check environment variables

See if environment variables are interpolated as expected.

```
export DATABASE=postgresql 
sudo \
DB_TEST_USERNAME=root DB_TEST_PASSWORD=yourpassword \
docker-compose -f ../docker-source/docker-compose.yml up \
config
```

## Override the frontend config.json 

Place your custom `config.json` file in this location before running the container. 

```
$HOST_VOLUMES/app-data/
frontend/classes/static/assets/data/config.json
```


## Building images

```
# Run these in gsrs-ci/<service>

# You may want to run with --build-arg RELEASE_MODE=$RELEASE_MODE

# ==== 

export DOCKER_SOURCE=../../docker-source

cd gsrs-ci

cd substances
docker build -f $DOCKER_SOURCE/substances/Dockerfile --no-cache --progress=plain  --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-substances:0.0.1-SNAPSHOT .

cd ..
cd gateway
docker build -f $DOCKER_SOURCE/gateway/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-gateway:0.0.1-SNAPSHOT .

cd ..
cd frontend
export FRONTEND_TAG='development_3.0'
# export FRONTEND_TAG='GSRSv3.1.1PUB'
docker build -f $DOCKER_SOURCE/frontend/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-frontend:0.0.1-SNAPSHOT .

cd ..
cd adverse-events
docker build -f $DOCKER_SOURCE/adverse-events/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-adverse-events:0.0.1-SNAPSHOT .

cd ..
cd applications
docker build -f $DOCKER_SOURCE/applications/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-applications:0.0.1-SNAPSHOT .

cd ..
cd clinical-trials
docker build -f $DOCKER_SOURCE/clinical-trials/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-clinical-trials:0.0.1-SNAPSHOT .

cd ..
cd impurities
docker build -f $DOCKER_SOURCE/impurities/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-impurities:0.0.1-SNAPSHOT .

cd ..
cd invitro-pharmacology
docker build -f $DOCKER_SOURCE/invitro-pharmacology/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-invitro-pharmacology:0.0.1-SNAPSHOT .

cd ..
cd products
docker build -f $DOCKER_SOURCE/products/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-products:0.0.1-SNAPSHOT .

cd ..
cd ssg4m
docker build -f $DOCKER_SOURCE/ssg4m/Dockerfile --no-cache --progress=plain --build-arg BUILD_VERSION=$BUILD_VERSION -t gsrs3/gsrs-emb-docker-ssg4m:0.0.1-SNAPSHOT .
```

## Create/reset database init.sql files

```
# Volumes/db/<flavor>/init folder
cd project 
tar -xvzf db.init.sql.tar.gz

```

## Clean up indexes

Before committing to Git, clean up folders from test instances

```
rm -r ./volumes/app-data/adverse-events/ginas.ix
rm -r ./volumes/app-data/applications/ginas.ix
rm -r ./volumes/app-data/clinical-trials/ginas.ix
rm -r ./volumes/app-data/impurities/ginas.ix
rm -r ./volumes/app-data/invitro-pharmacology/ginas.ix
rm -r ./volumes/app-data/substances/ginas.ix
```

## Wipe databases

```
rm -r ./volumes/app-data/db/mariadb/info && mkdir -p ./volumes/app-data/db/mariadb/info
rm -r ./volumes/app-data/db/postgresql/info && mkdir -p ./volumes/app-data/db/postgresql/info
rm -r ./volumes/app-data/db/mysql/info && mkdir -p ./volumes/app-data/db/mysql/info
```

## Find more files to exclude from commits or clean up

```
find . -type f  | grep -v app-data/db | grep -v 'frontend/classes'  | grep -v gsrs-ci
```

# Backup init.sql scripts 

```
tar -cvzf db.init.sql.tar.gz  $(find volumes -name init -type d)
```

# Backup configuration files

```
tar -cvzf flavor.env-db.conf.tar.gz  $(find volumes -type f -name  "*env-db.conf")
tar -cvzf backup.all.conf.tar.gz  $(find volumes -name conf -type d) 
```
