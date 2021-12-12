FROM maven:3.6.1-jdk-8-alpine

WORKDIR /app

COPY /demo /app

RUN mvn package -Dskiptest && mv target/demo-0.0.1-SNAPSHOT.jar /run/worker.jar && rm -rf * 

EXPOSE 8080

CMD java -jar /run/worker.jar
