FROM alpine/git as clone
WORKDIR /app
RUN git clone https://github.com/IAC2-Project/iacmf

FROM maven:3.8.5-openjdk-18 as build
WORKDIR /app
COPY --from=clone /app/iacmf /app
RUN mvn clean package -DskipTests
EXPOSE 9091
ENTRYPOINT ["java","-jar","/app/target/framework-0.0.1-SNAPSHOT.jar"]