
FROM alpine:latest AS build

ARG RELEASE_MODE='public'
ENV TZ=America/New_York

COPY . .
RUN if [ -f settings.xml ]; then \
  mkdir -p /root/.m2; \
  cp settings.xml /root/.m2/settings.xml; \ 
fi && \
apk add --no-cache git openjdk17 && \
if [ -f "installExtraJars.sh" ]; then sh installExtraJars.sh; fi && \
./mvnw clean -U package -Dwith.packaging=jar -DskipTests && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
mkdir -p /app/data/gateway  && \
mkdir -p /app/data/gateway/conf  && \
mkdir -p /app/data/gateway/classes

WORKDIR /app/data/gateway
EXPOSE 9081
ENTRYPOINT ["java", "-cp", "/target/gateway.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]

