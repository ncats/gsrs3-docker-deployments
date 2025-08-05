
FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ARG TZ=America/New_York
ENV TZ=${TZ}

COPY . .

RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \ 
fi && \
apk add --no-cache git openjdk17 && \
rm -rf src/main/resources/discovery-env.conf && \
rm -rf src/main/resources/discovery.conf && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
sh ./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/discovery  && \
mkdir -p /app/data/discovery/conf  && \
mkdir -p /app/data/discovery/classes

WORKDIR /app/data/discovery
EXPOSE 8761
ENTRYPOINT ["java", "-cp", "/target/discovery.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
