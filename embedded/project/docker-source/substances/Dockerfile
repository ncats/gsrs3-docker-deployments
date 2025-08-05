
FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG STARTER_MODULE_BRANCH=master
ARG SUBSTANCES_MODULE_BRANCH=master
ARG TZ=America/New_York
ENV TZ=${TZ}
# Set headless mode to avoid AWT GUI errors in POI/iText
ENV JAVA_TOOL_OPTIONS="-Djava.awt.headless=true"

COPY . .
RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \
fi && \
apk add --no-cache \
    git \
    openjdk17 \
    fontconfig \
    ttf-dejavu \
    && fc-cache -fv && \
if [ "$RELEASE_MODE" == "development" ]; then \
    git clone --branch $STARTER_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-starter.git && \
    git clone --branch $SUBSTANCES_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-substances.git && \
    cd gsrs-spring-starter &&  ./mvnw clean -U install -DskipTests && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    ./mvnw clean -U install -DskipTests && \
    cd .. && cd gsrs-spring-module-substances && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    ./mvnw clean -U install -DskipTests && \
    cd .. ; \
fi && \
rm -rf src/main/resources/substances-env.conf && \
rm -rf src/main/resources/substances-env-db.conf && \
rm -rf src/main/resources/substances.conf && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/substances/conf && \
mkdir -p /app/data/substances/classes

WORKDIR /app/data/substances
EXPOSE 8080
ENTRYPOINT ["java", "-cp", "/target/substances.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
