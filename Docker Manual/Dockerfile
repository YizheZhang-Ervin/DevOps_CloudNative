FROM openjdk:8-jdk-slim
LABEL maintainer=ervinzhang
COPY target/*.jar   /app.jar
ENTRYPOINT ["java","-jar","/app.jar"]