FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG STARTER_MODULE_BRANCH=master
ARG SUBSTANCES_MODULE_BRANCH=master
ARG PRODUCTS_MODULE_BRANCH=starter
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
  echo "PRODUCTS_MODULE_BRANCH: "$PRODUCTS_MODULE_BRANCH && \
  git clone --branch $STARTER_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-starter.git && \
  git clone --branch $SUBSTANCES_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-substances.git && \
  git clone --branch $PRODUCTS_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-drug-products.git && \
  cd gsrs-spring-starter && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    sh ./mvnw clean -U install -DskipTests && \
  cd ../gsrs-spring-module-substances && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    sh ./mvnw clean -U install -DskipTests && \
  cd ../gsrs-spring-module-drug-products && \
    if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
    sh ./mvnw clean -U install -DskipTests && \
  cd ..; \
fi && \
rm -rf src/main/resources/products-env.conf && \
rm -rf src/main/resources/products-env-db.conf && \
rm -rf src/main/resources/products.conf && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
sh ./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/products/conf && \
mkdir -p /app/data/products/classes

WORKDIR /app/data/products
EXPOSE 8084
ENTRYPOINT ["java", "-cp", "/target/products.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
