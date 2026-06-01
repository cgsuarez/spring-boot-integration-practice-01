# Airport Management API

API REST para la gestión de aeropuertos, vuelos y pasajeros, construida con **Spring Boot 3.2**, **Spring Data JPA** y probada con **JUnit 5** y **MockMvc**.

## Tecnologías

- **Java 17**
- **Spring Boot 3.2** (Web, Data JPA, Validation)
- **Maven** como gestor de dependencias
- **H2** — base de datos en memoria para desarrollo
- **PostgreSQL 16** — base de datos para integración y producción
- **Docker Compose** — para levantar PostgreSQL localmente
- **Testcontainers** — para pruebas de integración con PostgreSQL
- **JUnit 5 + MockMvc** — pruebas unitarias y de integración

## Estructura del proyecto

```
src/main/java/ec/edu/epn/
├── AirportApplication.java          # Clase principal
├── model/
│   ├── Airport.java                 # Entidad: aeropuerto
│   ├── Flight.java                  # Entidad: vuelo
│   └── Passenger.java               # Entidad: pasajero
├── repository/
│   ├── AirportRepository.java       # Repositorio JPA — aeropuertos
│   ├── FlightRepository.java        # Repositorio JPA — vuelos
│   └── PassengerRepository.java     # Repositorio JPA — pasajeros
├── service/
│   ├── AirportService.java          # Lógica de negocio — aeropuertos
│   ├── FlightService.java           # Lógica de negocio — vuelos
│   └── PassengerService.java        # Lógica de negocio — pasajeros
├── controller/
│   ├── AirportController.java       # Endpoints REST — /api/airports
│   ├── FlightController.java        # Endpoints REST — /api/flights
│   └── PassengerController.java     # Endpoints REST — /api/passengers
├── dto/
│   ├── AirportRequest.java          # DTO para crear/actualizar aeropuertos
│   ├── FlightRequest.java           # DTO para crear/actualizar vuelos
│   └── PassengerRequest.java        # DTO para crear/actualizar pasajeros
└── exception/
    ├── ResourceNotFoundException.java
    └── GlobalExceptionHandler.java  # Manejador global de excepciones
```

## Modelo de datos

### Airport (Aeropuerto)
| Campo   | Tipo   | Descripción                  |
|---------|--------|------------------------------|
| id      | Long   | Identificador (autogenerado) |
| name    | String | Nombre del aeropuerto        |
| code    | String | Código IATA (3 caracteres)   |
| city    | String | Ciudad donde se ubica        |
| country | String | País donde se ubica          |

### Flight (Vuelo)
| Campo         | Tipo          | Descripción                  |
|---------------|---------------|------------------------------|
| id            | Long          | Identificador (autogenerado) |
| flightNumber  | String        | Número de vuelo (único)      |
| origin        | Airport       | Aeropuerto de origen         |
| destination   | Airport       | Aeropuerto de destino        |
| departureTime | LocalDateTime | Fecha/hora de salida         |
| arrivalTime   | LocalDateTime | Fecha/hora de llegada        |
| status        | String        | Estado (SCHEDULED, DELAYED, CANCELLED, COMPLETED, IN_FLIGHT) |

### Passenger (Pasajero)
| Campo          | Tipo   | Descripción                  |
|----------------|--------|------------------------------|
| id             | Long   | Identificador (autogenerado) |
| firstName      | String | Nombre                       |
| lastName       | String | Apellido                     |
| email          | String | Correo electrónico (único)   |
| passportNumber | String | Número de pasaporte (único)  |

### Relaciones
- Un **Airport** tiene muchos vuelos como origen (`departures`) y como destino (`arrivals`).
- Un **Flight** pertenece a un aeropuerto de origen y a uno de destino.
- Un **Flight** puede tener muchos **Passenger** (relación `@ManyToMany` a través de la tabla `flight_passengers`).

## Endpoints REST

### Aeropuertos — `/api/airports`

| Método   | Ruta                    | Descripción                        |
|----------|-------------------------|------------------------------------|
| `POST`   | `/api/airports`         | Crear un aeropuerto                |
| `GET`    | `/api/airports`         | Listar todos los aeropuertos       |
| `GET`    | `/api/airports/{id}`    | Buscar aeropuerto por ID           |
| `GET`    | `/api/airports/code/{code}` | Buscar aeropuerto por código IATA |
| `GET`    | `/api/airports/search?city=X&country=Y` | Buscar por ciudad o país |
| `PUT`    | `/api/airports/{id}`    | Actualizar aeropuerto              |
| `DELETE` | `/api/airports/{id}`    | Eliminar aeropuerto                |

### Vuelos — `/api/flights`

| Método   | Ruta                                      | Descripción                      |
|----------|--------------------------------------------|----------------------------------|
| `POST`   | `/api/flights`                             | Crear un vuelo                   |
| `GET`    | `/api/flights`                             | Listar todos los vuelos          |
| `GET`    | `/api/flights/{id}`                        | Buscar vuelo por ID              |
| `GET`    | `/api/flights/number/{flightNumber}`       | Buscar vuelo por número          |
| `GET`    | `/api/flights/status/{status}`             | Buscar vuelos por estado         |
| `GET`    | `/api/flights/between?start=X&end=Y`       | Buscar vuelos entre fechas       |
| `PUT`    | `/api/flights/{id}`                        | Actualizar vuelo                 |
| `POST`   | `/api/flights/{flightId}/passengers/{passengerId}` | Agregar pasajero al vuelo |
| `DELETE` | `/api/flights/{flightId}/passengers/{passengerId}` | Remover pasajero del vuelo |
| `DELETE` | `/api/flights/{id}`                        | Eliminar vuelo                   |

### Pasajeros — `/api/passengers`

| Método   | Ruta                                   | Descripción                     |
|----------|----------------------------------------|---------------------------------|
| `POST`   | `/api/passengers`                      | Crear un pasajero               |
| `GET`    | `/api/passengers`                      | Listar todos los pasajeros      |
| `GET`    | `/api/passengers/{id}`                 | Buscar pasajero por ID          |
| `GET`    | `/api/passengers/email/{email}`        | Buscar pasajero por email       |
| `GET`    | `/api/passengers/passport/{passport}`  | Buscar pasajero por pasaporte   |
| `PUT`    | `/api/passengers/{id}`                 | Actualizar pasajero             |
| `DELETE` | `/api/passengers/{id}`                 | Eliminar pasajero               |

## Perfiles de Spring

El proyecto utiliza perfiles de Spring Boot para configurar la base de datos según el entorno:

| Perfil        | Base de datos | Propósito                                 |
|---------------|---------------|-------------------------------------------|
| `dev`         | H2 en memoria | Desarrollo local rápido                   |
| `integration` | PostgreSQL    | Pruebas de integración (requiere Docker)  |
| `prod`        | PostgreSQL    | Producción (requiere Docker)              |

### Ejecutar con perfil específico

```bash
# Perfil dev (por defecto, H2 en memoria)
mvn spring-boot:run

# Perfil integration (requiere docker compose up -d primero)
mvn spring-boot:run -Dspring-boot.run.profiles=integration

# Perfil prod
mvn spring-boot:run -Dspring-boot.run.profiles=prod
```

## Ejecutar el proyecto

### Requisitos previos

- Java 17+
- Maven 3.8+
- Docker y Docker Compose (solo para perfiles `integration` y `prod`)

### Compilar

```bash
mvn clean compile
```

### Ejecutar en modo desarrollo (H2)

```bash
mvn spring-boot:run
```

La consola H2 estará disponible en: [http://localhost:8080/h2-console](http://localhost:8080/h2-console)

### Ejecutar con PostgreSQL (Docker)

```bash
# 1. Levantar PostgreSQL
docker compose up -d

# 2. Ejecutar con perfil integration
mvn spring-boot:run -Dspring-boot.run.profiles=integration

# 3. Detener PostgreSQL al terminar
docker compose down
```

## Ejecutar pruebas

```bash
# Pruebas unitarias (Surefire, clases *Test.java)
mvn test

# Pruebas de integración (Failsafe, clases *IT.java)
mvn verify

# Solo una prueba de integración específica
mvn verify -Dit.test=AirportControllerIT
```

---

# Tarea: Implementación de pruebas de integración con MockMvc

## Objetivo

Implementar pruebas de integración para los controladores REST usando **Spring Boot Test**, **MockMvc** y **JUnit 5**. Las pruebas deben validar el comportamiento completo de cada endpoint (HTTP request → Controller → Service → Repository → HTTP response).

## Instrucciones

### 1. Configuración del entorno de pruebas

Crea el archivo `src/test/resources/application.properties` con una configuración de base de datos H2 en memoria para que las pruebas no dependan de infraestructura externa:

```properties
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
```

### 2. Clases de prueba a implementar

Debes crear las siguientes clases en `src/test/java/ec/edu/epn/integration/`:

| Clase                       | Controlador a probar     |
|-----------------------------|--------------------------|
| `AirportControllerIT.java`  | `AirportController`      |
| `FlightControllerIT.java`   | `FlightController`       |
| `PassengerControllerIT.java`| `PassengerController`    |

Usa el sufijo `IT` (Integration Test) para que Maven las distinga de las pruebas unitarias.

### 3. Estructura base de cada prueba

Cada clase de prueba debe seguir esta estructura:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class AirportControllerIT {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    // Pruebas aquí...
}
```

### 4. Casos de prueba requeridos

Para cada controlador, implementa al menos los siguientes casos de prueba:

#### AirportControllerIT
- [ ] `shouldCreateAirport` — Crear un aeropuerto y verificar HTTP 201 + datos en la respuesta
- [ ] `shouldRejectDuplicateAirportCode` — Intentar crear dos aeropuertos con el mismo código IATA
- [ ] `shouldFindAllAirports` — Listar todos los aeropuertos (crea 2 antes)
- [ ] `shouldFindAirportById` — Buscar por ID y verificar los datos
- [ ] `shouldFindAirportByCode` — Buscar por código IATA
- [ ] `shouldReturn404WhenAirportNotFound` — Buscar un ID inexistente → HTTP 404
- [ ] `shouldUpdateAirport` — Actualizar y verificar los cambios
- [ ] `shouldDeleteAirport` — Eliminar y verificar que ya no existe (GET → 404)
- [ ] `shouldRejectInvalidAirportRequest` — Enviar datos inválidos y verificar HTTP 400

#### FlightControllerIT
- [ ] `shouldCreateFlight` — Crear un vuelo (necesitas crear aeropuertos primero en `@BeforeEach`)
- [ ] `shouldRejectDuplicateFlightNumber` — Intentar crear dos vuelos con el mismo número
- [ ] `shouldRejectArrivalBeforeDeparture` — Validar que la llegada no sea antes de la salida
- [ ] `shouldFindAllFlights` — Listar todos los vuelos
- [ ] `shouldFindFlightById` — Buscar por ID
- [ ] `shouldFindFlightByFlightNumber` — Buscar por número de vuelo
- [ ] `shouldFindFlightsByStatus` — Filtrar por estado
- [ ] `shouldFindFlightsBetweenDates` — Buscar vuelos en un rango de fechas
- [ ] `shouldUpdateFlight` — Actualizar estado del vuelo
- [ ] `shouldReturn404WhenFlightNotFound` — Flight inexistente → HTTP 404
- [ ] `shouldDeleteFlight` — Eliminar y verificar que ya no existe

#### PassengerControllerIT
- [ ] `shouldCreatePassenger` — Crear un pasajero y verificar HTTP 201
- [ ] `shouldRejectDuplicateEmail` — Intentar crear dos pasajeros con el mismo email
- [ ] `shouldFindAllPassengers` — Listar todos los pasajeros
- [ ] `shouldFindPassengerById` — Buscar por ID
- [ ] `shouldFindPassengerByEmail` — Buscar por email
- [ ] `shouldFindPassengerByPassportNumber` — Buscar por número de pasaporte
- [ ] `shouldReturn404WhenPassengerNotFound` — Pasajero inexistente → HTTP 404
- [ ] `shouldUpdatePassenger` — Actualizar datos del pasajero
- [ ] `shouldDeletePassenger` — Eliminar y verificar que ya no existe
- [ ] `shouldRejectInvalidEmail` — Enviar email inválido y verificar HTTP 400

### 5. Criterios de evaluación

| Criterio                    | Puntos |
|-----------------------------|--------|
| Todas las pruebas compilan  | 2.0    |
| Todas las pruebas pasan     | 3.0    |
| Uso correcto de `MockMvc` y `ObjectMapper` | 1.0 |
| Uso de `@BeforeEach` para preparar datos | 0.5 |
| Cobertura de casos de error (404, 400, 409) | 1.5 |
| Verificación de cuerpo de respuesta con `jsonPath()` | 1.0 |
| Organización del código (helpers privados) | 1.0 |
| **Total**                   | **10.0** |

### 6. Recursos útiles

- [Documentación de MockMvc](https://docs.spring.io/spring-framework/reference/testing/spring-mvc-test-framework.html)
- [Spring Boot Test — @AutoConfigureMockMvc](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html#testing.spring-boot-applications.with-mock-environment)
- [Hamcrest Matchers para jsonPath](http://hamcrest.org/JavaHamcrest/javadoc/2.2/)
- [Guía de JUnit 5](https://junit.org/junit5/docs/current/user-guide/)

### 7. Entregables

1. Las tres clases de prueba en `src/test/java/ec/edu/epn/integration/`
2. Archivo `src/test/resources/application.properties`
3. Captura de pantalla del resultado de `mvn verify` mostrando todas las pruebas en verde

---

## Licencia

Proyecto académico — Escuela Politécnica Nacional — Validación de Software — 2026A
