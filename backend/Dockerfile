# ==============
# Build Stage
# ==============
FROM gradle:8.14.3-jdk21 AS build

WORKDIR /app

COPY build.gradle settings.gradle ./
COPY gradle ./gradle
COPY gradlew ./

COPY src ./src

RUN chmod +x ./gradlew
RUN ./gradlew clean bootJar -x test

# ===============
# Run Stage
# ===============
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=build /app/build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT [ "java", "-jar", "app.jar" ]