FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG STARTER_MODULE_BRANCH=master
ARG SUBSTANCES_MODULE_BRANCH=master
ARG APPLICATIONS_MODULE_BRANCH=starter

ENV TZ=America/New_York

COPY . .
RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \
fi && \
apk add --no-cache git openjdk17 && \
if [ "$RELEASE_MODE" = "development" ]; then \
  git clone --branch $STARTER_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-starter.git && \
  git clone --branch $SUBSTANCES_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-substances.git && \
  git clone --branch $APPLICATIONS_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-drug-applications.git && \
  cd gsrs-spring-starter &&  ./mvnw clean -U install -DskipTests && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    ./mvnw clean -U install -DskipTests && \
  cd .. && cd gsrs-spring-module-substances && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    ./mvnw clean -U install -DskipTests && \
  cd .. && cd gsrs-spring-module-drug-applications && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    ./mvnw clean -U install -DskipTests && \
  cd ..; \
fi && \
rm -rf src/main/resources/applications-env.conf && \
rm -rf src/main/resources/applications-env-db.conf && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/applications/conf && \
mkdir -p /app/data/applications/classes

WORKDIR /app/data/applications
EXPOSE 8084
ENTRYPOINT ["java", "-cp", "/target/applications.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
