# Docker mvn build
FROM maven:3.6.3-jdk-11 AS build

ARG RELEASE_MODE='public'
ARG STARTER_MODULE_BRANCH=master
ARG SUBSTANCES_MODULE_BRANCH=master
ARG APPLICATIONS_MODULE_BRANCH=starter

COPY . .
RUN if [ "$RELEASE_MODE" = "development" ]; then \
  git clone --branch $STARTER_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-starter.git && \
  git clone --branch $SUBSTANCES_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-substances.git && \
  git clone --branch $APPLICATIONS_MODULE_BRANCH --depth 2 https://github.com/ncats/gsrs-spring-module-drug-applications.git && \
  cd gsrs-spring-starter &&  mvn clean -U install -DskipTests && \ 
    sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh && \
    mvn clean -U install -DskipTests && \
  cd .. && cd gsrs-spring-module-substances && \
    sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh && \
    mvn clean -U install -DskipTests && \
   cd .. && cd gsrs-spring-module-drug-applications && \ 
    # sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh \
    mvn clean -U install -DskipTests; \
fi

RUN rm -rf src/main/resources/application.conf && \
    rm -rf src/main/resources/applications-env.conf && \
    rm -rf src/main/resources/applications-env-db.conf && \
    sed -i.bak -e "s/\.\/mvnw/mvn/g" installExtraJars.sh && sh installExtraJars.sh && \
    mvn clean -U package -Dwith.packaging=jar -DskipTests

ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mkdir -p /app/data/applications/conf
RUN mkdir -p /app/data/applications/classes

WORKDIR /app/data/applications
EXPOSE 8084
ENTRYPOINT ["java", "-cp", "/target/applications.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
