# Docker mvn build
FROM maven:3.6.3-jdk-11 AS build

ARG RELEASE_MODE='public'
ARG STARTER_MODULE_BRANCH=master
ARG SUBSTANCES_MODULE_BRANCH=master
ARG INVITRO_PHARMACOLOGY_MODULE_BRANCH=master

COPY . .
RUN if [ "$RELEASE_MODE" = "development" ]; then \
  git clone --branch $STARTER_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-starter.git && \
  git clone --branch $SUBSTANCES_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-substances.git && \
  git clone --branch $INVITRO_PHARMACOLOGY_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-invitro-pharmacology.git && \
  cd gsrs-spring-starter &&  mvn clean -U install -DskipTests && \ 
    sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh && \
    mvn clean -U install -DskipTests && \
  cd .. && cd gsrs-spring-module-substances && \
    sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh && \
    mvn clean -U install -DskipTests && \
  cd .. && cd gsrs-spring-module-invitro-pharmacology && \ 
    # sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh \
    mvn clean -U install -DskipTests; \
fi

RUN rm -rf src/main/resources/application.conf && \
    rm -rf src/main/resources/invitro-pharmacology-env.conf && \
    rm -rf src/main/resources/invitro-pharmacology-env-db.conf && \
    sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh && \
    mvn clean -U package -Dwith.packaging=jar -DskipTests

ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mkdir -p /app/data/invitro-pharmacology/conf
RUN mkdir -p /app/data/invitro-pharmacology/classes

WORKDIR /app/data/invitro-pharmacology
EXPOSE 8084
ENTRYPOINT ["java", "-cp", "/target/invitro-pharmacology.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]

