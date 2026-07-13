
FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG SERVICES_COMMON_BRANCH=master
ARG FRONTEND_TAG=development_3.0
ARG USE_VOLUME_DIST='no'
ARG TZ=America/New_York
ENV TZ=${TZ}

COPY . .

# if USE_VOLUME_DIST=='yes' then please copy the contents of
# the GSRSFrontend/dist/browser in: $HOST_VOLUMES/app-data/frontend/classes/static
# and adjust your config.json file in: assets/data/config.json

# Current only USE_VOLUME_DIST==yes works. 
# However note for later ..
# If you the second alternative, you can still place your custom config.json in:
# $HOST_VOLUMES/app-data/frontend/classes/static/assets/data/config.json
# and it will override the one that is downloaded from Github.
# However, if you previously use the first way, then you need to delete your copy 
# of the frontend distribution. 

RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \
fi && \
apk update && \
apk add --no-cache bash gnupg git openjdk17 libgcc libstdc++ && \
if [ "$RELEASE_MODE" == "development" ]; then \
    if [ -n "$SERVICES_COMMON_BRANCH" ]; then \
        git clone --branch $SERVICES_COMMON_BRANCH https://github.com/ncats/gsrs-services-common.git; \
        cd gsrs-services-common;  \
        sh mvnw clean -U install -DskipTests; \
        cd ..; \
    fi; \
fi && \
rm -rf src/main/resources/frontend-env.conf && \
rm -rf src/main/resources/frontend.conf && \
# if [ "${USE_VOLUME_DIST}" = 'yes' ]; then \

#    echo "Using local deployment binaries"; \
#    sh ./mvnw clean -U package -Dwith.packaging=jar \
#      -Dfrontend.tag=local_deployable_binaries \
#      -Dfrontend.repo=file:///app/data/frontend/binaries \
#      -Dnode.disable -Dwithout.static \
#      -Dwithout.visualizer  -DskipTests; \
# else \

   echo "Using Angular source from Git"; \
   sh ./mvnw clean -U package -Dwith.packaging=jar \
   -Dfrontend.tag=$FRONTEND_TAG \
   -Dwithout.visualizer  -DskipTests; \
# fi && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/frontend/conf && \
mkdir -p /app/data/frontend/classes/static

WORKDIR /app/data/frontend
EXPOSE 8082
ENTRYPOINT ["java", "-cp", "/target/frontend.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
