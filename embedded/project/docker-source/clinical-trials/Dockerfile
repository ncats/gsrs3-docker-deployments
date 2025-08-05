
FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG STARTER_MODULE_BRANCH=master
ARG SUBSTANCES_MODULE_BRANCH=master
ARG CLINICAL_TRIALS_MODULE_BRANCH=master
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
  echo "STARTER_MODULE_BRANCH: "$STARTER_MODULE_BRANCH && \
  echo "SUBSTANCES_MODULE_BRANCH: "$SUBSTANCES_MODULE_BRANCH && \
  echo "CLINICAL_TRIALS_MODULE_BRANCH: "$CLINICAL_TRIALS_MODULE_BRANCH && \
  git clone --branch $STARTER_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-starter.git && \
  git clone --branch $SUBSTANCES_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-substances.git && \
  git clone --branch $CLINICAL_TRIALS_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-clinical-trials.git && \
  cd gsrs-spring-starter && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    sh ./mvnw clean -U install -DskipTests && \
  cd ../gsrs-spring-module-substances && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    sh ./mvnw clean -U install -DskipTests && \
  cd ../gsrs-spring-module-clinical-trials && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    sh ./mvnw clean -U install -DskipTests && \
  cd ..; \
fi && \
rm -rf src/main/resources/clinical-trials-env.conf && \
rm -rf src/main/resources/clinical-trials-env-db.conf && \
rm -rf src/main/resources/clinical-trials.conf && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/clinical-trials/conf && \
mkdir -p /app/data/clinical-trials/classes

WORKDIR /app/data/clinical-trials
EXPOSE 8089
ENTRYPOINT ["java", "-cp", "/target/clinical-trials.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
