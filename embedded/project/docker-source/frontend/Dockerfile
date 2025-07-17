# Docker mvn build
FROM maven:3.6.3-jdk-11 AS build
ARG FRONTEND_TAG=development_3.0

COPY . .
ADD settings.xml /root/.m2/settings.xml
RUN whoami && cat  /root/.m2/settings.xml

# RUN echo "Getting linux distribution release information:\n====" && \
#  cat /etc/os-release && echo "\n====\n" 

# RUN apt update && apt install -y python3 

# If you use this you can put the a build of the Angular in /app/data/frontend/classes/static
RUN mvn clean -U package -Dwith.packaging=jar -Dnode.disable -Dwithout.static -Dfrontend.tag=$FRONTEND_TAG -Dwithout.visualizer  -DskipTests

# RUN mvn clean -U package -Dwith.packaging=jar -Dfrontend.tag=$FRONTEND_TAG -Dwithout.visualizer  -DskipTests

ENV TZ=America/New_York
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mkdir -p /app/data/frontend/conf
RUN mkdir -p /app/data/frontend/classes/static
WORKDIR /app/data/frontend
EXPOSE 8082
ENTRYPOINT ["java", "-cp", "/target/frontend.jar:classes:conf", "org.springframework.boot.loader.JarLauncher"]
