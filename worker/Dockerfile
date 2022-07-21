FROM maven:3.8.6-eclipse-temurin-18-alpine

WORKDIR /app

COPY . .

RUN mvn package -DskiptTests && \
    mv target/worker-jar-with-dependencies.jar /run/worker.jar && \
    rm -rf /app/*

CMD java -jar /run/worker.jar
