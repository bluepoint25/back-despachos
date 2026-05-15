# back-Despachos_SpringBoot

Microservicio de gestión de despachos para **Innovatech Chile**.  
Desarrollado con **Spring Boot 3.4.4 + Java 17 + MySQL**.

---

## Tecnologías

- Java 17
- Spring Boot 3.4.4
- Spring Data JPA
- MySQL 8
- Docker (multi-stage build)
- GitHub Actions (CI/CD)

---

## Estructura del repositorio

```
back-Despachos_SpringBoot/
├── src/                        # Código fuente Java
├── pom.xml                     # Dependencias Maven
├── Dockerfile                  # Imagen Docker multi-stage
└── .github/
    └── workflows/
        └── cicd-despachos.yml  # Pipeline CI/CD
```

---

## Dockerfile — Decisiones técnicas

Se implementó **multi-stage build** con dos etapas:

| Stage | Imagen base | Propósito |
|-------|-------------|-----------|
| `builder` | `maven:3.9.6-eclipse-temurin-17` | Compilar el JAR |
| runtime | `eclipse-temurin:17-jre-alpine` | Ejecutar el JAR (imagen mínima) |

**Buenas prácticas aplicadas:**
- Usuario no root (`appuser`) → mínimo privilegio
- Solo JRE en producción, no JDK completo → imagen más liviana
- `COPY pom.xml` antes que el código fuente → cache de capas de Maven
- `UseContainerSupport` → JVM respeta los límites de memoria del contenedor

---

## Variables de entorno requeridas

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `SPRING_DATASOURCE_URL` | URL JDBC de MySQL | `jdbc:mysql://mysql:3306/innovatech_db` |
| `SPRING_DATASOURCE_USERNAME` | Usuario de BD | `appuser` |
| `SPRING_DATASOURCE_PASSWORD` | Contraseña de BD | `password` |
| `SERVER_PORT` | Puerto del servicio | `8081` |

---

## Cómo ejecutar localmente

```bash
# Desde la raíz del proyecto (donde está docker-compose.yml)
docker compose up --build back-despachos
```

El servicio queda disponible en: `http://localhost:8081`

---

## Pipeline CI/CD

El pipeline se activa automáticamente al hacer **push en la rama `deploy`**.

### Flujo completo:

```
push a rama deploy
        │
        ▼
1. Checkout del código
        │
        ▼
2. Configurar credenciales AWS (desde GitHub Secrets)
        │
        ▼
3. Login en Amazon ECR
        │
        ▼
4. docker build → docker push a ECR
        │
        ▼
5. AWS SSM → EC2 Backend:
   - docker pull (imagen nueva)
   - docker stop / docker rm (contenedor anterior)
   - docker run (nuevo contenedor)
```

### GitHub Secrets requeridos:

| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | Credencial AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS Academy |
| `AWS_SESSION_TOKEN` | Token de sesión AWS Academy |
| `AWS_REGION` | Región AWS (ej: `us-east-1`) |
| `ECR_REGISTRY` | URL base del ECR |
| `ECR_REPO_URL_DESPACHOS` | URL del repositorio ECR |
| `EC2_BACKEND_INSTANCE_ID` | ID de la instancia EC2 |
| `SPRING_DATASOURCE_URL` | URL JDBC |
| `MYSQL_USER` | Usuario MySQL |
| `MYSQL_PASSWORD` | Contraseña MySQL |

---

## Commits

Este repositorio utiliza commits descriptivos con el formato:

```
feat: descripción de nueva funcionalidad
fix:  descripción de corrección
docs: cambios en documentación
ci:   cambios en pipeline
```
"# back-despachos" 
