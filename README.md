# Back Despachos — API REST

API REST para la gestión de **despachos / órdenes de entrega**. Forma parte del sistema junto con el [frontend](../front_despacho) y la [API de Ventas](../back-Ventas_SpringBoot): cuando una venta genera su despacho, este servicio almacena y administra esa orden de entrega.

Desarrollada con **Spring Boot 3.4.4** y **Java 17**.

> El proyecto Maven se encuentra dentro de la carpeta [`Springboot-API-REST-DESPACHO/`](Springboot-API-REST-DESPACHO). En la raíz del repositorio está la configuración de despliegue (`k8s/`).

---

## 🛠️ Stack tecnológico

| Categoría        | Tecnología |
|------------------|------------|
| Lenguaje         | Java 17 |
| Framework        | Spring Boot 3.4.4 |
| Web              | Spring Web (REST) |
| Persistencia     | Spring Data JPA / Hibernate |
| Base de datos    | MySQL (producción) · H2 (disponible) |
| Validación       | Spring Boot Starter Validation |
| Documentación    | springdoc OpenAPI (Swagger UI) |
| Utilidades       | Lombok |
| Build            | Maven (`mvnw` incluido) |

---

## 📦 Modelo de datos — `Despacho`

| Campo              | Tipo        | Notas |
|--------------------|-------------|-------|
| `idDespacho`       | `Long`      | PK, autogenerado |
| `fechaDespacho`    | `LocalDate` | Formato `yyyy-MM-dd` |
| `patenteCamion`    | `String`    | Patente del camión asignado |
| `intento`          | `int`       | N.º de intentos de entrega |
| `idCompra`         | `Long`      | ID de la venta asociada |
| `direccionCompra`  | `String`    | Dirección de entrega |
| `valorCompra`      | `Long`      | Valor de la compra |
| `despachado`       | `boolean`   | Estado de entrega (por defecto `false`) |

---

## 🔌 Endpoints

Base: `/api/v1/despachos` · CORS habilitado para cualquier origen (`@CrossOrigin` + `CorsConfig`).

| Método   | Ruta                        | Descripción                       | Respuesta |
|----------|-----------------------------|-----------------------------------|-----------|
| `GET`    | `/api/v1/despachos`         | Lista todos los despachos         | `200 OK` |
| `GET`    | `/api/v1/despachos/{id}`    | Obtiene un despacho por ID        | `200 OK` / `404` |
| `POST`   | `/api/v1/despachos`         | Crea un nuevo despacho            | `201 Created` |
| `PUT`    | `/api/v1/despachos/{id}`    | Actualiza un despacho existente   | `200 OK` / `404` |
| `DELETE` | `/api/v1/despachos/{id}`    | Elimina un despacho               | `204 No Content` / `404` |

### Ejemplo de cuerpo (`POST` / `PUT`)

```json
{
  "fechaDespacho": "2024-02-05",
  "patenteCamion": "ABCD-12",
  "intento": 0,
  "idCompra": 1,
  "direccionCompra": "Av. Siempre Viva 742",
  "valorCompra": 22990,
  "despachado": false
}
```

### 📖 Documentación interactiva (Swagger)

Con la aplicación corriendo:

```
http://localhost:8081/swagger-ui.html
```

---

## ⚙️ Configuración

La conexión a la base de datos se define con **variables de entorno** (ver `src/main/resources/application.properties`):

| Variable      | Descripción                  |
|---------------|------------------------------|
| `DB_ENDPOINT` | Host del servidor MySQL      |
| `DB_PORT`     | Puerto (ej. `3306`)          |
| `DB_NAME`     | Nombre de la base de datos   |
| `DB_USERNAME` | Usuario                      |
| `DB_PASSWORD` | Contraseña                   |

> Hibernate usa `ddl-auto=update`. El servicio escucha en el puerto **8081** (`server.port=8081`).

---

## 🚀 Ejecución local

Requisitos: **JDK 17** y MySQL.

```bash
cd Springboot-API-REST-DESPACHO

# Definir variables de entorno de la BD (ejemplo)
export DB_ENDPOINT=localhost DB_PORT=3306 DB_NAME=despachos \
       DB_USERNAME=root DB_PASSWORD=secret

# Ejecutar con el wrapper de Maven
./mvnw spring-boot:run          # Linux / macOS
mvnw.cmd spring-boot:run        # Windows
```

### Compilar y empaquetar

```bash
./mvnw clean package            # genera target/*.jar
java -jar target/*.jar
```

### Tests

```bash
./mvnw test
```

---

## 🐳 Docker

Build multi-etapa (Maven para compilar, JRE Alpine para ejecutar).

```bash
cd Springboot-API-REST-DESPACHO
docker build -t back-despacho .
docker run -p 8081:8081 \
  -e DB_ENDPOINT=... -e DB_PORT=3306 -e DB_NAME=despachos \
  -e DB_USERNAME=... -e DB_PASSWORD=... \
  back-despacho
```

---

## ☸️ Kubernetes

El manifiesto `k8s/deployment.yaml` define un Deployment (`back-despacho`, 2 réplicas) y un Service tipo `LoadBalancer`.

```bash
kubectl apply -f k8s/deployment.yaml
```

---

## 📂 Estructura

```
back-Despachos_SpringBoot/
├── Springboot-API-REST-DESPACHO/
│   ├── src/main/java/com/citt/
│   │   ├── SpringbootApiRestDespachoApplication.java
│   │   ├── config/
│   │   │   ├── OpenApiConfig.java
│   │   │   └── CorsConfig.java
│   │   ├── controller/DespachoController.java
│   │   ├── exceptions/            # Manejo de errores (404, etc.)
│   │   └── persistence/
│   │       ├── entity/Despacho.java
│   │       ├── repository/DespachoRepository.java
│   │       └── services/          # DespachoService + impl
│   ├── src/main/resources/application.properties
│   ├── src/test/                  # Pruebas
│   ├── Dockerfile
│   └── pom.xml
└── k8s/deployment.yaml
```
