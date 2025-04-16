#base image
FROM openjdk:17
#Working directory
WORKDIR /app
#copy from source to destination
COPY target/hello-world-1.0-SNAPSHOT.jar app.jar
#run
ENTRYPOINT ["java", "-jar", "app.jar"]

