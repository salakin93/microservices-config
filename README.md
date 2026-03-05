# Microservices Configuration Repository

Este repositorio contiene la **configuración centralizada de todos los microservicios** de la plataforma.

Las configuraciones son consumidas por el **Config Server (`config-service`)** utilizando **Spring Cloud Config Server**, permitiendo que los microservicios obtengan su configuración desde una fuente única y versionada.

---

# Propósito del repositorio

En una arquitectura de microservicios, cada servicio necesita configuraciones como:

* puertos
* credenciales de base de datos
* URLs de otros servicios
* configuraciones de seguridad
* propiedades específicas del dominio

En lugar de duplicar estas configuraciones dentro de cada microservicio, se almacenan en este repositorio y son entregadas dinámicamente por el **Config Server**.

Esto permite:

* centralizar configuraciones
* modificar configuraciones sin recompilar servicios
* gestionar configuraciones por ambiente
* versionar cambios de configuración

---

# Arquitectura

```text
Microservices
      │
      ▼
Config Service (Spring Cloud Config Server)
      │
      ▼
Repositorio de configuraciones (este repositorio)
```

Flujo de funcionamiento:

1. Un microservicio inicia.
2. El microservicio solicita su configuración al **Config Server**.
3. El Config Server obtiene la configuración desde este repositorio.
4. El Config Server devuelve la configuración al microservicio.

---

# Estructura del repositorio

```text
microservices-config
│
├── identity-service.properties
├── gateway-service.properties
├── discovery-service.properties
├── config-service.properties
├── document-service.properties
└── ai-service.properties
```

Cada archivo representa la configuración de un microservicio específico.

El nombre del archivo **debe coincidir con `spring.application.name` del microservicio**.

---

# Configuración de servicios

## identity-service.properties

Configuración del servicio de identidad y autenticación.

```properties
server.port=8081
spring.application.name=identity-service

spring.datasource.url=jdbc:postgresql://postgres:5432/identity_db
spring.datasource.username=postgres
spring.datasource.password=postgres

spring.jpa.hibernate.ddl-auto=update
spring.jpa.open-in-view=false
```

---

## gateway-service.properties

Configuración del API Gateway.

```properties
server.port=8080
spring.application.name=gateway-service
```

---

## discovery-service.properties

Configuración del servicio de descubrimiento (Eureka Server).

```properties
server.port=8761
spring.application.name=discovery-service
```

---

## config-service.properties

Configuración del servidor de configuración.

```properties
server.port=8888
spring.application.name=config-service

spring.cloud.config.server.git.uri=https://github.com/salakin93/microservices-config
spring.cloud.config.server.git.clone-on-start=true
spring.cloud.config.server.git.default-label=main
```

---

## document-service.properties

Configuración del servicio de documentos.

```properties
server.port=8082
spring.application.name=document-service

spring.datasource.url=jdbc:postgresql://postgres:5432/document_db
spring.datasource.username=postgres
spring.datasource.password=postgres

spring.jpa.hibernate.ddl-auto=update
spring.jpa.open-in-view=false

file.storage.path=/data/uploads
```

---

## ai-service.properties

Configuración del servicio de inteligencia artificial.

```properties
server.port=8083
spring.application.name=ai-service

chatpdf.base-url=https://api.chatpdf.com/v1
chatpdf.api-key=${CHATPDF_API_KEY:}

management.endpoints.web.exposure.include=health,info
```

---

# Convenciones de configuración

Para mantener consistencia en todos los microservicios se siguen estas convenciones:

| Propiedad               | Descripción                              |
| ----------------------- | ---------------------------------------- |
| server.port             | Puerto donde se ejecuta el microservicio |
| spring.application.name | Identificador del servicio               |
| spring.datasource.*     | Configuración de base de datos           |
| spring.jpa.*            | Configuración de JPA/Hibernate           |

---

# Convención de nombres

El nombre del archivo debe coincidir con:

```text
spring.application.name
```

Ejemplo:

| Servicio         | Archivo                     |
| ---------------- | --------------------------- |
| identity-service | identity-service.properties |
| gateway-service  | gateway-service.properties  |
| document-service | document-service.properties |

---

# Cómo los microservicios obtienen su configuración

Cada microservicio debe incluir en su configuración local:

```properties
spring.config.import=configserver:http://config-service:8888
```

y

```properties
spring.application.name=<nombre-del-servicio>
```

Ejemplo para identity-service:

```properties
spring.application.name=identity-service
spring.config.import=configserver:http://config-service:8888
```

Cuando el servicio arranca:

1. Se conecta a `config-service`
2. Solicita su configuración
3. `config-service` la obtiene desde este repositorio

---

# Configuración por ambiente (futuro)

Spring Cloud Config permite manejar múltiples ambientes:

```text
identity-service-dev.properties
identity-service-prod.properties
identity-service-test.properties
```

Ejemplo de endpoint:

```text
http://config-service:8888/identity-service/dev
```

Esto permite tener configuraciones distintas para:

* desarrollo
* staging
* producción

---

# Seguridad

Este repositorio **no debe contener secretos sensibles** como:

* API keys
* contraseñas de producción
* tokens

Las credenciales sensibles deben gestionarse mediante:

* variables de entorno
* secret managers
* vaults
* configuración del entorno de despliegue

---

# Integración con Docker

Todos los microservicios se ejecutarán dentro de contenedores Docker y obtendrán su configuración desde el Config Server.

Ejemplo de conexión en Docker Compose:

```yaml
environment:
  SPRING_CONFIG_IMPORT: configserver:http://config-service:8888
```

El nombre `config-service` funciona como DNS interno dentro de Docker Compose.

---

# Estado actual

✔ Configuraciones base definidas
✔ Compatible con Spring Cloud Config Server
✔ Preparado para Docker Compose
✔ Escalable para múltiples ambientes

---

# Licencia

Proyecto académico / investigación.
