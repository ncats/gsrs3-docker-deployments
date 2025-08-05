
FROM alpine:latest AS build

ARG FRONTEND_TAG=development_3.0
ARG USE_VOLUME_DIST='yes'
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
apk add --no-cache git openjdk17 && \
rm -rf src/main/resources/frontend-env.conf && \
rm -rf src/main/resources/frontend.conf && \
if [ "${USE_VOLUME_DIST}" = 'yes' ]; then \
    sh ./mvnw clean -U package -Dwith.packaging=jar -Dnode.disable -Dwithout.static -Dfrontend.tag=$FRONTEND_TAG -Dwithout.visualizer  -DskipTests; \
else \
    sh ./mvnw clean -U package -Dwith.packaging=jar -Dfrontend.tag=$FRONTEND_TAG -Dwithout.visualizer  -DskipTests; \
fi && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/frontend/conf && \
mkdir -p /app/data/frontend/classes/static

WORKDIR /app/data/frontend
EXPOSE 8082
ENTRYPOINT ["java", "-cp", "/target/frontend.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
