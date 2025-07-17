
FROM alpine:latest AS build

COPY . .

RUN whoami && cat  /root/.m2/settings.xml

RUN ./mvnw clean -U package -Dwith.packaging=jar  -DskipTests

ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mkdir -p /app/discovery
RUN mkdir -p /app/data/discovery/conf
RUN mkdir -p /app/data/discovery/classes
WORKDIR /app/data/discovery
EXPOSE 8761
ENTRYPOINT ["java", "-cp", "/target/discovery.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
