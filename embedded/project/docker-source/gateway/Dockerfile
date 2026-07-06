
FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG SERVICES_COMMON_BRANCH=master
ARG TZ=America/New_York
ENV TZ=${TZ}

COPY . .

RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \ 
fi && \
apk add --no-cache git openjdk17 && \
if [ "$RELEASE_MODE" == "development" ]; then \
    if [ -n "$SERVICES_COMMON_BRANCH" ]; then \
        git clone --branch $SERVICES_COMMON_BRANCH https://github.com/ncats/gsrs-services-common.git; \
        cd gsrs-services-common;  \
        sh mvnw clean -U install -DskipTests; \
        cd ..; \
    fi; \
fi && \
rm -rf src/main/resources/gateway-env.conf && \
rm -rf src/main/resources/gateway.conf && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
sh ./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/gateway  && \
mkdir -p /app/data/gateway/conf  && \
mkdir -p /app/data/gateway/classes

WORKDIR /app/data/gateway
EXPOSE 8081
ENTRYPOINT ["java", "-cp", "/target/gateway.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
