# Asynchronous Notification System

Distributed asynchronous notification system with a microservices-based architecture, using Java/Spring Boot, Apache Kafka and MySQL.

## Overview

This is the **infrastructure repository** that orchestrates all system components via Docker Compose. The system implements an asynchronous alert processing flow composed of two independent microservices:

1. **notification-api** - Receives alerts via REST API and publishes them to Kafka
2. **alert-processor** - Consumes alerts from Kafka, processes them (simulated 500ms delay) and persists them in MySQL

## Repository Structure

This project is divided into **3 separate repositories**:

```
📦 Complete System
│
├── 📁 notification-api (Repository 1)
│   └── Microservice that receives alerts via REST and publishes to Kafka
│       URL: https://github.com/Dyel-L/notification-api
│
├── 📁 alert-processor (Repository 2)
│   └── Microservice that consumes from Kafka and persists to MySQL
│       URL: https://github.com/Dyel-L/alert-processor
│
└── 📁 infra-notification-system (Repository 3 - THIS)
    └── Docker Compose that orchestrates the entire infrastructure
        - Kafka + Zookeeper
        - MySQL
        - Microservice images (Docker Hub)
```

- ✅ **Separation of concerns**: Each microservice can evolve independently
- ✅ **Independent CI/CD**: Each service can have its own pipeline
- ✅ **Isolated versioning**: Changes in one service don't affect the others
- ✅ **Facilitates deployment**: Each service can be deployed separately
- ✅ **Umbrella repository**: Single entry point to bring up the whole stack

## Links to Repositories

### Application Repositories

- **notification-api**: [https://github.com/Dyel-L/notification-api](https://github.com/Dyel-L/notification-api)
  - Source code for the API microservice
  - Unit and integration tests

- **alert-processor**: [https://github.com/Dyel-L/alert-processor](https://github.com/Dyel-L/alert-processor)
  - Source code for the processor microservice
  - Unit tests

- **infra-notification-system**: [https://github.com/Dyel-L/infra-notification-system](https://github.com/Dyel-L/infra-notification-system) **(THIS REPOSITORY)**
  - Docker Compose and orchestration
  - Infrastructure documentation

## Technologies Used

### Applications
- **Java 17** - Programming language
- **Spring Boot 3.5.7** - Framework for microservices
- **Spring Kafka** - Kafka integration
- **Spring Data JPA** - Data persistence
- **Maven** - Dependency and build management
- **Lombok** - Boilerplate reduction

### Infrastructure
- **Apache Kafka 7.5.0** - Message broker for asynchronous communication
- **Zookeeper** - Kafka cluster coordination
- **MySQL 8.0** - Relational database
- **Docker** - Containerization
- **Docker Compose** - Container orchestration
- **Redis** - Distributed cache for deduplication of duplicate alerts

### Testing
- **JUnit 5** - Test framework
- **Mockito** - Mocks for unit tests
- **Spring Boot Test** - Integration testing

## Prerequisites

- **Docker 20.10+**
- **Docker Compose 2.0+**

## How to Run

### Start

```bash
# 1. Clone this repository
git clone https://github.com/Dyel-L/infra-notification-system.git

cd infra-notification-system

# 2. Bring the entire infrastructure up
docker-compose up -d
```

## ⚠️ Important: Docker Desktop

### Windows and macOS

**Before running any Docker commands, make sure Docker Desktop is open and running.**

### Linux

On Linux, just ensure the Docker service is active.

Docker will:
1. Automatically pull the images from Docker Hub
2. Start Zookeeper, Kafka and Redis
3. Start MySQL and create the database `alerts_db`
4. Start the two microservices

### Check Status

```bash
# Show status of all containers
docker-compose ps

# Expected result:
NAME               IMAGE                             COMMAND                  SERVICE            CREATED          STATUS                    PORTS
alert-processor    dyelll/alert-processor:latest     "java -jar app.jar"      alert-processor    39 seconds ago   Up 27 seconds
kafka              confluentinc/cp-kafka:7.5.0       "/etc/confluent/dock…"   kafka              39 seconds ago   Up 38 seconds             0.0.0.0:9092->9092/tcp, [::]:9092->9092/tcp
mysql              mysql:8.0                         "docker-entrypoint.s…"   mysql              39 seconds ago   Up 38 seconds (healthy)   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp
notification-api   dyelll/notification-api:latest    "java -jar app.jar"      notification-api   39 seconds ago   Up 27 seconds             0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp
redis              redis:7-alpine                    "docker-entrypoint.s…"   redis              39 seconds ago   Up 38 seconds (healthy)   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp
zookeeper          confluentinc/cp-zookeeper:7.5.0   "/etc/confluent/dock…"   zookeeper          39 seconds ago   Up 38 seconds             0.0.0.0:2181->2181/tcp, [::]:2181->2181/tcp
```

### Follow Logs

```bash
# See logs of all services
docker-compose logs -f

# See logs of a specific service
docker-compose logs -f notification-api
docker-compose logs -f alert-processor

# See only the last 100 lines
docker-compose logs --tail=100 -f
```

## Testing the System

## Available Endpoints

| Service          | Endpoint | Method | Port | Description |
|------------------|----------|--------|-------|-------------|
| notification-api | `/alerts` | POST | 8080 | Create a new alert |
| MySQL            | - | - | 3306 | Database |
| Kafka            | - | - | 9092 | Message broker |
| Zookeeper        | - | - | 2181 | Kafka coordination |
| Redis            | - | - | 6379 | Cache |

### Payload for the /alerts Endpoint

```json
{
  "alertType": "SECURITY", // REQUIRED
  "clientId": "´123", // REQUIRED
  "message": "Intrusion detected in sector 7",  // REQUIRED
  "severity": "MEDIUM", // REQUIRED
  "source": "Camera-01"
}
```
The `severity` field accepts the values: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`.

### 1️⃣ Send an Alert

Postman collection: [Postman link](https://www.postman.com/altimetry-engineer-56943415/desafio-ubisafe/collection/g61oww7/desafio-ubisafe?action=share&creator=41177636)

Or use the curl command below:

```bash
curl -X POST http://localhost:8080/alerts \
  -H "Content-Type: application/json" \
  -d '{
    "alertType": "EMAIL",
    "clientId": "123",
    "message": "Intrusion detected in sector 5",
    "severity": "MEDIUM",
    "source": "Camera-01"
}'
```

**Expected response (202 Accepted):**
```json
{
  "message": "Alert received and queued for processing",
  "id": "983d554e-9279-4490-9c63-65ebf40f6776",
  "status": "ACCEPTED"
}
```

**Example error response (500 Internal Server Error):**
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred",
  "timestamp": "2025-11-17T12:34:56.789Z",
  "status": 500
}
```

### 2️⃣ Verify Processing

```bash
# View processor logs
docker-compose logs -f alert-processor

# You will see logs like:
# alert-processor | Processing alert for clientId: 12345
# alert-processor | Alert processed successfully with status: PROCESSADO
```

### 3️⃣ Verify in the Database

```bash
# Connect to MySQL
docker exec -it mysql mysql -u root -proot alerts_db

# Query the alerts
SELECT * FROM alerts ORDER BY id DESC LIMIT 10;

# Exit MySQL
exit;
```

## Useful Commands
### Monitoring

```bash
# Show resource usage
docker stats

# Show processes running inside a container
docker top notification-api

# Inspect a container
docker inspect notification-api

# Show network
docker network inspect infra-notification-system_ubisafe-network
```

## Architectural Decisions

The system is composed of two independent microservices that communicate asynchronously through Apache Kafka:

```
Client → [notification-api] → Kafka (alerts topic) → [alert-processor] → MySQL
                ↓                                              ↓
              Redis                                     Failure Records
         (Deduplication)
```

### Key Characteristics

- Asynchronous alert processing
- Automatic deduplication (configurable window)
- Delivery guarantees with Kafka
- Transactional persistence with robust failure handling
- Full traceability of success and errors

---

## Microservices

### notification-api (Producer)

Responsibilities:
- Receive HTTP requests to create alerts
- Validate input payload (Bean Validation)
- Check for duplication using Redis
- Publish message to Kafka topic `alerts`
- Respond immediately with HTTP 202 (Accepted)

### alert-processor (Consumer)

Responsibilities:
- Consume messages from the `alerts` topic
- Deserialize and validate alerts
- Process with simulated delay (500ms)
- Persist processed alerts in MySQL
- Record failures in an independent transaction

---

### 1. Asynchronous Communication with Kafka

Why?
- ✅ **Decoupling**: API and Processor don't know about each other
- ✅ **Resilience**: If the processor goes down, messages remain in Kafka
- ✅ **Scalability**: Possible to add multiple processor instances
- ✅ **Performance**: API responds immediately (202) without waiting for processing
- ✅ **Delivery guarantee**: Kafka ensures messages are not lost

Producer configuration:
```java
configProps.put(ProducerConfig.ACKS_CONFIG, "all");
configProps.put(ProducerConfig.RETRIES_CONFIG, 3);
configProps.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 1);
```

- `acks=all`: Waits for confirmation from all in-sync brokers
- `retries=3`: Retries up to 3 times on failure
- `max.in.flight=1`: Ensures message ordering

Consumer configuration:
```java
configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.RECORD);
```

- Manual offset commit only after full processing
- Messages in error can be reprocessed or sent to a DLT

---

### 2. Deduplication with Redis

Implementation:
```java
public boolean isDuplicate(String alertId) {
    String key = PREFIX + alertId;
    Boolean firstTime = redisTemplate.opsForValue()
        .setIfAbsent(key, "1", windowSeconds, TimeUnit.SECONDS);
    return firstTime == null || !firstTime;
}
```

Why?
- ✅ Prevents duplicate processing of identical alerts within a short time window
- ✅ Redis in-memory is extremely fast
- ✅ Automatic TTL: keys expire after the configured window
- ✅ SETNX (Set If Not Exists) operation is atomic

Deduplication criteria:
- Same combination of `clientId + alertType + message + severity`
- `timestamp` and `source` are not considered
- Deterministic hash ensures the same ID for identical alerts

---

### 3. Producer-Consumer Pattern

notification-api (Producer):
- Single responsibility: validate and publish
- Does not know who will process the message
- Responds quickly to the client

alert-processor (Consumer):
- Single responsibility: process and persist
- Does not know who sent the alert
- Processes at its own pace

---

### 4. Simulated Delay (500ms)

Implementation:
```java
private static final long PROCESSING_DELAY_MS = 500;

Thread.sleep(PROCESSING_DELAY_MS);
```

Rationale:
- Simulates real processing (sending emails, external validations, etc.)
- Demonstrates the benefit of asynchronous processing
- The client receives 202 immediately, without waiting for the 500ms
- Makes the flow easier to visualize during demos

---

### 5. Transactional Persistence

#### Infrastructure Layer (Kafka Listener)

The listener consumes the message and delegates to the application service. It should not carry transaction responsibility nor contain business logic.

```java
@KafkaListener(topics = "alerts", groupId = "processor-group")
public void consumeAlert(String alertJson) {
    // No @Transactional – just orchestrates the flow
    alertService.processAlert(alertJson);
}
```

#### Application Layer (Services)

Responsible for processing, mapping and persisting. Separates the main flow and failure recording into distinct services.

Success flow – Single transaction:
```java
@Transactional
public AlertEntity processAlert(String alertJson) {
    Alert alert = objectMapper.readValue(alertJson, Alert.class);
    AlertEntity entity = alertMapper.toSuccessEntity(alert);
    return alertRepository.save(entity);
}
```

Failure flow – Independent transaction:
```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void registerFailureFromAlertJson(String alertJson, String failureReason) {
    Alert alert = objectMapper.readValue(alertJson, Alert.class);
    AlertEntity failedEntity = alertMapper.toFailureEntityFromAlert(alert, failureReason);
    alertRepository.save(failedEntity);
}
```

Why?
- Keeps transactional boundaries clear
- Ensures failures are recorded even if the main transaction rolls back

---

### 6. Exception Handling

Hierarchy:
```
AlertProcessingException (base)
    └── InvalidAlertJsonException (malformed JSON)
```

Strategy:
```java
try {
    alertService.processAlert(alertJson);
} catch (InvalidAlertJsonException e) {
    // Invalid JSON → save raw payload
    alertFailureService.registerFailureFromRawPayload(alertJson, reason);
    throw e;
} catch (AlertProcessingException e) {
    // Business/technical error → save alert data
    alertFailureService.registerFailureFromAlertJson(alertJson, reason);
    throw e;
}
```

---

### 7. Dedicated Mapper

```java
@Component
public class AlertMapper {
    public AlertEntity toSuccessEntity(Alert alert) { ... }
    public AlertEntity toFailureEntityFromAlert(Alert alert, String reason) { ... }
    public AlertEntity toFailureEntityFromRawPayload(String payload, String reason) { ... }
}
```

Why?
- Avoids code duplication
- Facilitates maintenance and evolution
- Clear separation of responsibilities
- Easier isolated testing

---

## Transactional Flows

### Success Flow

1. Kafka listener receives the message
2. `alertService.processAlert()` starts transaction **T1**
3. Deserializes JSON → maps entity → persists to MySQL
4. If everything succeeds:
- Commit **T1**
- Kafka confirms the offset
5. If a failure occurs:
- Automatic rollback of **T1**
- Offset not confirmed → message will be reprocessed

### Failure Flow

1. An error occurs in the listener or service (invalid JSON, MySQL failure, etc.)
2. `alertFailureService.registerFailure*()` opens a new transaction **T2** (`REQUIRES_NEW`)
3. Failure log is saved in MySQL
4. Commit **T2**, independent from **T1**
5. Exception rethrown → rollback of **T1**
6. Offset not confirmed → message will be reprocessed or sent to a DLT

---

## Architecture Benefits

- ✅ **Data integrity:** either processed successfully or failure is recorded
- ✅ **Independent transactions:** rollback of main flow does not remove failure logs
- ✅ **Separation of responsibilities:** listener only orchestrates; services do the heavy work
- ✅ **Resilience:** `REQUIRES_NEW` ensures failure registration even when the main flow fails
- ✅ **Traceability:** failures are stored with timestamp and detailed reason
- ✅ **Scalability:** Kafka distributes load among consumers; multiple instances possible
- ✅ **Testability:** services are decoupled and injectable; mapper can be tested independently

---

## Applied Principles

| Principle | Application |
|-----------|-------------|
| **Single Responsibility** | Each component has a single clear responsibility |
| **Open/Closed** | Easy to extend without modifying existing code |
| **Dependency Inversion** | Dependencies via interfaces (Repository, ObjectMapper) |
| **Separation of Concerns** | Well-defined layers (infra, application, domain) |
| **Fail-Fast** | Validations early in the flow |
| **Defensive Programming** | Robust exception handling |

### 8. Alert Status

Each processed alert has a final status:

- **`SUCCESS`**: Processing completed successfully
- **`FAILURE`**: Error during processing

### 9. Use of Docker Images

Strategy:
- Microservice images published on Docker Hub
- Facilitates deployment and distribution
- No need to build locally
- Automatic download of images

### 10. Separation into 3 Repositories

Benefits:
- Each service evolves independently
- Isolated CI/CD per service
- Easier maintenance and versioning
- Umbrella repository as a single entry point

### Docker Images

- `dyelll/notification-api:latest` - [Docker Hub](https://hub.docker.com/r/dyelll/notification-api)
- `dyelll/alert-processor:latest` - [Docker Hub](https://hub.docker.com/r/dyelll/alert-processor)


## 👥 Author

Developed for the Ubisafe Challenge - Asynchronous Notification System

Dylan Bitencourt Gonçalves

---

**Status:** ✅ Ready for use

**Last updated:** November 2025
```