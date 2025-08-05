
FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG STARTER_MODULE_BRANCH=master
ARG SUBSTANCES_MODULE_BRANCH=master
ARG SSG4M_MODULE_BRANCH=master
ARG TZ=America/New_York
ENV TZ=${TZ}

COPY . .
RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \
fi && \
apk add --no-cache git openjdk17 && \
if [ "$RELEASE_MODE" = "development" ]; then \
  echo "RELEASE_MODE: "$RELEASE_MODE && \
  echo "SSG4M_MODULE_BRANCH: "$SSG4M_MODULE_BRANCH && \
  git clone --branch $SSG4M_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-ssg4.git && \
  cd gsrs-spring-module-ssg4 && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    sh ./mvnw clean -U install -DskipTests && \
  cd ..; \
fi && \
rm -rf src/main/resources/ssg4m-env.conf && \
rm -rf src/main/resources/ssg4m-env-db.conf && \
rm -rf src/main/resources/ssg4m.conf && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/ssg4m/conf && \
mkdir -p /app/data/ssg4m/classes

WORKDIR /app/data/ssg4m
EXPOSE 8088
ENTRYPOINT ["java", "-cp", "/target/ssg4m.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
