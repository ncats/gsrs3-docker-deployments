
FROM alpine:latest AS build

ARG FRONTEND_TAG=development_3.0
ARG USE_DEPLOYABLE_BINARY='yes'

ENV TZ=America/New_York

# RUN echo "Getting linux distribution release information:\n====" && \
#  cat /etc/os-release && echo "\n====\n"
# python3

COPY . .
RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \
fi && \
apk add --no-cache git openjdk17 && \
if [ "${USE_DEPLOYABLE_BINARY}" = 'yes' ]; then \
    ./mvnw clean -U package -Dwith.packaging=jar -Dnode.disable -Dwithout.static -Dfrontend.tag=$FRONTEND_TAG -Dwithout.visualizer  -DskipTests; \
else \
    ./mvnw clean -U package -Dwith.packaging=jar -Dfrontend.tag=$FRONTEND_TAG -Dwithout.visualizer  -DskipTests; \
fi && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/frontend/conf && \
mkdir -p /app/data/frontend/classes/static

WORKDIR /app/data/frontend
EXPOSE 8082
ENTRYPOINT ["java", "-cp", "/target/frontend.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
