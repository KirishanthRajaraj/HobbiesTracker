# Stage 1: build with Maven (use JDK 21)
FROM maven:3.9.9-eclipse-temurin-21 AS builder
WORKDIR /app

# Copy only pom to cache dependencies
COPY pom.xml ./
# If you have a mvnw, copy it and make executable:
# COPY mvnw ./
# COPY .mvn .mvn
RUN mvn -B -DskipTests dependency:go-offline

# Copy sources and build
COPY src ./src
RUN mvn -B -DskipTests package

# Stage 2: runtime image (slim JRE)
FROM eclipse-temurin:21-jre AS runtime

# Install curl for healthcheck
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

WORKDIR /app
# Copy the built jar (assumes single jar in target/)
COPY --from=builder /app/target/*.jar app.jar

# Expose port your app uses
EXPOSE 8080

# Recommended: set runtime java options via env var
ENV JAVA_OPTS="-Xms256m -Xmx512m -Djava.security.egd=file:/dev/./urandom"

# Use non-root user
USER appuser

# If you use Spring Boot Actuator and want Docker healthchecks, enable them in compose or here (commented)
# HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
#   CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -jar /app/app.jar"]
