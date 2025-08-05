# Notes for GSRS Docker embedded deployment

## Terminal Environment

```
cd gsrs3-docker-deployments/embedded

export embedded_root_dir=$(pwd)



export gsrs_ci_dir=$embedded_root_dir/project/gsrs-ci
# or ...
# export gsrs_ci_dir=$embedded_root_dir/project/gsrs3-main-deployment

export DOCKER_SOURCE=$embedded_root_dir/project/docker-source
export HOST_VOLUMES=$embedded_root_dir/project/volumes

export DB_TEST_USERNAME=root
export DB_TEST_PASSWORD=yourpassword

# development|public
export RELEASE_MODE=public

export BUILD_VERSION=v2025.0429.1

# As needed, only use if in development mode
export STARTER_MODULE_BRANCH='master'
export SUBSTANCES_MODULE_BRANCH='master'
export ADVERSE_EVENTS_MODULE_BRANCH='starter'
export APPLICATIONS_MODULE_BRANCH='starter'
export CLINICAL_TRIALS_MODULE_BRANCH='master'
export IMPURITIES_MODULE_BRANCH='starter'
export INVITRO_PHARMACOLOGY_MODULE_BRANCH='master'
export PRODUCTS_MODULE_BRANCH='starter'
export SSG4M_MODULE_BRANCH='master'
```

## Purpose

This Docker recipe is mainly meant for local testing and also to provide an introduction to using Docker with GSRS in an embedded Tomcat scenario.

## gsrs-ci

Below "gsrs-ci" refers to a deployments folder used by FDA. but you may use any similar deployment repository/folder:

- gsrs-ci
- gsrs-example-deployment
- gsrs3-main-deployment

## Clone gsrs-ci

```
# Temporarily clone your gsrs-ci repo here

cd gsrs3-docker-deployments/embedded/project
clone http://github.com/ncats/gsrs-ci.git
cd gsrs-ci 

```

## Running the containers

First you'll need to build your images (see below)

```
# use ONE of the database flavors:

- h2 (DATABASE="" in this case)
- postgresql
- mariadb
- mysql

# The docker-compose.yml file should require one of these but does not yet do so.

cd gsrs-ci

export DATABASE=mariadb
DB_TEST_USERNAME=root DB_TEST_PASSWORD=yourpassword \
docker-compose -f $DOCKER_SOURCE/docker-compose.yml up \
$DATABASE frontend gateway substances products

# If you need to use sudo, put the sudo before the db credentials.
export DATABASE=postgresql
sudo \
DB_TEST_USERNAME=root DB_TEST_PASSWORD=yourpassword \
docker-compose -f $DOCKER_SOURCE/docker-compose.yml up \
$DATABASE frontend gateway substances products
```

## Available services

```
adverse-events
applications
discovery
frontend
gateway 
clinical-trials
impurities
invitro-pharmacology
substances
ssg4m

# Most entity services, depend on the substances service for full functionality. 
# Discovery isn't used in practice, yet.
```

## Pick one database service

```
h2 
mariadb 
mysql 
postgresql

#  For h2, set $DATABASE='' on the docker-compose command. It is used by default.
```

## Database `<service>-env-db.conf` files and init files

These files contain default configs details for flavor

- blank.env-db.conf.tar.gz (use for h2)
- db.init.sql.tar.gz
- mariadb.env-db.conf.tar.gz
- mysql.env-db.conf.tar.gz
- postgresql.env-db.conf.tar.gz

## Check environment variables

See if environment variables are interpolated as expected.

```
export DATABASE=postgresql 
sudo \
DB_TEST_USERNAME=root DB_TEST_PASSWORD=yourpassword \
docker-compose -f $DOCKER_SOURCE/docker-compose.yml config
```

## Override the frontend config.json

Place your custom `config.json` file in this location before running the container.

```
$HOST_VOLUMES/app-data/frontend/classes/static/assets/data/config.json
```

## Building images

```
# Run these in the gsrs-ci/<service> corresponding folder

# Make sure you have set a value RELEASE_MODE (development|public). This will determine whether a `Dockerfile` looks for code in Github or Maven Central.

# A settings.xml file may be required, we are trying to make it no so.

# ==== 

cd gsrs-ci

cd substances
# cp ../../settings.xml . 
 docker build -f $DOCKER_SOURCE/substances/Dockerfile \
--platform linux/x86_64 \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg STARTER_MODULE_BRANCH=$STARTER_MODULE_BRANCH \
--build-arg SUBSTANCES_MODULE_BRANCH=$SUBSTANCES_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-substances:0.0.1-SNAPSHOT .

# On substances 
# --platform linux/x86_64  -- because got errors related to ehcache-failsafe.xml and Error loading shared library ld-linux-aarch64.so

# Experimental, may not work, not currently used by FDA
# not included in gsrs3-main-deployment
cd ..
cd discovery
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/discovery/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-discovery:0.0.1-SNAPSHOT .


cd ..
cd gateway
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/gateway/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-gateway:0.0.1-SNAPSHOT .

cd ..
cd frontend
# cp ../../settings.xml .
# export FRONTEND_TAG='development_3.0'
export FRONTEND_TAG='GSRSv3.1.2PUB'
docker build -f $DOCKER_SOURCE/frontend/Dockerfile \
--no-cache --progress=plain \
--build-arg FRONTEND_TAG=$FRONTEND_TAG \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-frontend:0.0.1-SNAPSHOT .

cd ..
cd adverse-events
cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/adverse-events/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg STARTER_MODULE_BRANCH=$STARTER_MODULE_BRANCH \
--build-arg SUBSTANCES_MODULE_BRANCH=$SUBSTANCES_MODULE_BRANCH \
--build-arg ADVERSE_EVENTS_MODULE_BRANCH=$ADVERSE_EVENTS_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-adverse-events:0.0.1-SNAPSHOT .


cd ..
cd applications
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/applications/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg STARTER_MODULE_BRANCH=$STARTER_MODULE_BRANCH \
--build-arg SUBSTANCES_MODULE_BRANCH=$SUBSTANCES_MODULE_BRANCH \
--build-arg APPLICATIONS_MODULE_BRANCH=$APPLICATIONS_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
 -t gsrs3/gsrs-emb-docker-applications:0.0.1-SNAPSHOT .

cd ..
cd clinical-trials
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/clinical-trials/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg STARTER_MODULE_BRANCH=$STARTER_MODULE_BRANCH \
--build-arg SUBSTANCES_MODULE_BRANCH=$SUBSTANCES_MODULE_BRANCH \
--build-arg CLINICAL_TRIALS_MODULE_BRANCH=$CLINICAL_TRIALS_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-clinical-trials:0.0.1-SNAPSHOT .

cd ..
cd impurities
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/impurities/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg STARTER_MODULE_BRANCH=$STARTER_MODULE_BRANCH \
--build-arg SUBSTANCES_MODULE_BRANCH=$SUBSTANCES_MODULE_BRANCH \
--build-arg IMPURITIES_MODULE_BRANCH=$IMPURITIES_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
 -t gsrs3/gsrs-emb-docker-impurities:0.0.1-SNAPSHOT .

cd ..
cd invitro-pharmacology
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/invitro-pharmacology/Dockerfile \
 --no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg STARTER_MODULE_BRANCH=$STARTER_MODULE_BRANCH \
--build-arg SUBSTANCES_MODULE_BRANCH=$SUBSTANCES_MODULE_BRANCH \
--build-arg INVITRO_PHARMACOLOGY_MODULE_BRANCH=$INVITRO_PHARMACOLOGY_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
 -t gsrs3/gsrs-emb-docker-invitro-pharmacology:0.0.1-SNAPSHOT .

cd ..
cd products
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/products/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg STARTER_MODULE_BRANCH=$STARTER_MODULE_BRANCH \
--build-arg SUBSTANCES_MODULE_BRANCH=$SUBSTANCES_MODULE_BRANCH \
--build-arg PRODUCTS_MODULE_BRANCH=$PRODUCTS_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-products:0.0.1-SNAPSHOT .

cd ..
cd ssg4m
# cp ../../settings.xml .
docker build -f $DOCKER_SOURCE/ssg4m/Dockerfile \
--no-cache --progress=plain \
--build-arg RELEASE_MODE=$RELEASE_MODE \
--build-arg SSG4M_MODULE_BRANCH=$SSG4M_MODULE_BRANCH \
--build-arg BUILD_VERSION=$BUILD_VERSION \
-t gsrs3/gsrs-emb-docker-ssg4m:0.0.1-SNAPSHOT .

```

## Create/reset database init.sql files

```

cd project 
tar -xvzf db.init.sql.tar.gz

```

## Clean up indexes

Before committing to Git, clean up folders from test instances

```
rm -r $HOST_VOLUMES/app-data/adverse-events/ginas.ix
rm -r $HOST_VOLUMES/app-data/applications/ginas.ix
rm -r $HOST_VOLUMES/app-data/clinical-trials/ginas.ix
rm -r $HOST_VOLUMES/app-data/impurities/ginas.ix
rm -r $HOST_VOLUMES/app-data/invitro-pharmacology/ginas.ix
rm -r $HOST_VOLUMES/app-data/products/ginas.ix
rm -r $HOST_VOLUMES/app-data/substances/ginas.ix
```

## Wipe databases

```

rm -r $HOST_VOLUMES/$app-data/db/mariadb/info && mkdir -p ./volumes/app-data/db/mariadb/info
rm -r $HOST_VOLUMES/app-data/db/postgresql/info && mkdir -p ./volumes/app-data/db/postgresql/info
rm -r  $HOST_VOLUMES/app-data/db/mysql/info && mkdir -p ./volumes/app-data/db/mysql/info
```

## Find more files to exclude from commits or clean up

```
find . -type f  | grep -v app-data/db | grep -v 'frontend/classes'  | grep -v gsrs-ci
```

## Backup init.sql scripts

```
tar -cvzf db.init.sql.tar.gz  $(find volumes -name init -type d)
```

## Backup configuration files from volumes

```
tar -cvzf flavor.env-db.conf.tar.gz  $(find volumes -type f -name  "*env-db.conf")

tar -cvzf  backup.volumes.confs.tar.gz  $(find volumes -name "*.conf" -type f) 
```

## Backup configuration files from a gsrs-ci deployment and put in volumes/app-data structure

```
if ( test -d temp.ci.confs ); then
  echo "Aborted temp.ci.confs folder already exists";
else
  rm -f temp.ci.confs.tar.gz
  for f in $(find . -name "*.conf" -type f); do
  dir="$(dirname "${f}")"
  file="$(basename "${f}")"
  newDir=$(echo "$dir" | sed -e 's/\.\/\(.*\)\/src\/main\/resources/temp.ci.confs\/volumes\/app-data/\/\1\/conf/g')
  mkdir -p $newDir
  cp $f $newDir/$file
  tar -cvzf temp.ci.confs.tar.gz temp.ci.confs
  done;
fi
```

## To do

```
Separate db init/info folders
Add depends on substances to all entity services in docker-compose.yml (except ssg4m) 
salt file?
```
