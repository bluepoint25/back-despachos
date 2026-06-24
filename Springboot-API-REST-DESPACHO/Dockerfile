# ================================
# STAGE 1: Build
# ================================
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

# Copiar archivos de dependencias primero (cache de capas)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copiar el código fuente y compilar
COPY src ./src
RUN mvn clean package -DskipTests -B

# ================================
# STAGE 2: Runtime (imagen mínima)
# ================================
FROM eclipse-temurin:17-jre-alpine

# Crear usuario no root por seguridad (mínimo privilegio)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copiar el JAR desde el stage de build
COPY --from=builder /app/target/*.jar app.jar

# Cambiar propietario al usuario no root
RUN chown appuser:appgroup app.jar

# Usar usuario no root
USER appuser

# Puerto expuesto por el microservicio de Despachos
EXPOSE 8081

# Healthcheck para verificar que el servicio está activo
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8081/actuator/health || exit 1

# Comando de inicio con opciones de memoria optimizadas para contenedor
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
