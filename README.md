# Sistema de Notificação Assíncrona - Ubisafe

Sistema distribuído de notificação assíncrona com arquitetura baseada em microsserviços, utilizando Java/Spring Boot, Apache Kafka e MySQL.

##  Visão Geral

Este é o **repositório de infraestrutura** que orquestra todos os componentes do sistema através do Docker Compose. O sistema implementa um fluxo assíncrono de processamento de alertas composto por dois microsserviços independentes:

1. **notification-api** - Recebe alertas via REST API e os publica no Kafka
2. **alert-processor** - Consome alertas do Kafka, processa (com delay simulado de 500ms) e persiste no MySQL

##  Estrutura dos Repositórios

Este projeto está dividido em **3 repositórios separados**:

```
📦 Sistema Completo
│
├── 📁 notification-api (Repositório 1)
│   └── Microsserviço que recebe alertas via REST e publica no Kafka
│       URL: https://github.com/Dyel-L/notification-api
│
├── 📁 alert-processor (Repositório 2)
│   └── Microsserviço que consome do Kafka e persiste no MySQL
│       URL: https://github.com/Dyel-L/alert-processor
│
└── 📁 infra-notification-system (Repositório 3 - ESTE)
    └── Docker Compose que orquestra toda a infraestrutura
        - Kafka + Zookeeper
        - MySQL
        - Imagens dos microsserviços (Docker Hub)
```



- ✅ **Separação de responsabilidades**: Cada microsserviço evolui independentemente
- ✅ **CI/CD independente**: Cada serviço pode ter seu próprio pipeline
- ✅ **Versionamento isolado**: Mudanças em um serviço não afetam o outro
- ✅ **Facilita deploy**: Cada serviço pode ser deployado separadamente
- ✅ **Repositório umbrella**: Ponto único para subir toda a stack

## 🔗 Links dos Repositórios

### Repositórios do Código Fonte

- **notification-api**: [https://github.com/Dyel-L/notification-api](https://github.com/Dyel-L/notification-api)
  - Código fonte do microsserviço de API
  - Testes unitários e de integração

- **alert-processor**: [https://github.com/Dyel-L/alert-processor](https://github.com/Dyel-L/alert-processor)
  - Código fonte do microsserviço processador
  - Testes unitários

- **infra-notification-system**: [https://github.com/Dyel-L/infra-notification-system](https://github.com/Dyel-L/infra-notification-system) **(ESTE REPOSITÓRIO)**
  - Docker Compose e orquestração
  - Documentação de infraestrutura

##  Tecnologias Utilizadas

### Aplicações
- **Java 17** - Linguagem de programação
- **Spring Boot 3.5.7** - Framework para microsserviços
- **Spring Kafka** - Integração com Apache Kafka
- **Spring Data JPA** - Persistência de dados
- **Maven** - Gerenciamento de dependências e build
- **Lombok** - Redução de boilerplate

### Infraestrutura
- **Apache Kafka 7.5.0** - Message broker para comunicação assíncrona
- **Zookeeper** - Coordenação do cluster Kafka
- **MySQL 8.0** - Banco de dados relacional
- **Docker** - Containerização
- **Docker Compose** - Orquestração de containers
- **Redis** - Cache distribuído para deduplicação de alertas duplicados

### Testes
- **JUnit 5** - Framework de testes
- **Mockito** - Mocks para testes unitários
- **Spring Boot Test** - Testes de integração

##  Pré-requisitos

- **Docker 20.10+**
- **Docker Compose 2.0+**

##  Como Executar

###  Início 



```bash
# 1. Clone este repositório
git clone https://github.com/Dyel-L/infra-notification-system.git

cd infra-notification-system

# 2. Suba toda a infraestrutura
docker-compose up -d
```
## ⚠️ Importante: Docker Desktop

### Windows e macOS

**Antes de executar qualquer comando Docker, certifique-se de que o Docker Desktop está aberto e rodando.**

### Linux

No Linux, basta garantir que o serviço está ativo

O Docker irá:
1. Baixar as imagens do Docker Hub automaticamente
2. Subir Zookeeper e Kafka e Redis
3. Subir MySQL e criar o banco `alerts_db`
4. Subir os dois microsserviços

###  Verificar Status

```bash
# Ver status de todos os containers
docker-compose ps

# Resultado esperado:
NAME               IMAGE                             COMMAND                  SERVICE            CREATED          STATUS                    PORTS
alert-processor    dyelll/alert-processor:latest     "java -jar app.jar"      alert-processor    39 seconds ago   Up 27 seconds
kafka              confluentinc/cp-kafka:7.5.0       "/etc/confluent/dock…"   kafka              39 seconds ago   Up 38 seconds             0.0.0.0:9092->9092/tcp, [::]:9092->9092/tcp
mysql              mysql:8.0                         "docker-entrypoint.s…"   mysql              39 seconds ago   Up 38 seconds (healthy)   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp
notification-api   dyelll/notification-api:latest    "java -jar app.jar"      notification-api   39 seconds ago   Up 27 seconds             0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp
redis              redis:7-alpine                    "docker-entrypoint.s…"   redis              39 seconds ago   Up 38 seconds (healthy)   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp
zookeeper          confluentinc/cp-zookeeper:7.5.0   "/etc/confluent/dock…"   zookeeper          39 seconds ago   Up 38 seconds             0.0.0.0:2181->2181/tcp, [::]:2181->2181/tcp
```

###  Acompanhar Logs

```bash
# Ver logs de todos os serviços
docker-compose logs -f

# Ver logs de um serviço específico
docker-compose logs -f notification-api
docker-compose logs -f alert-processor

# Ver apenas as últimas 100 linhas
docker-compose logs --tail=100 -f
```

##  Testando o Sistema

##  Endpoints Disponíveis

| Serviço          | Endpoint | Método | Porta | Descrição |
|------------------|----------|--------|-------|-----------|
| notification-api | `/alerts` | POST | 8080 | Criar novo alerta |
| MySQL            | - | - | 3306 | Banco de dados |
| Kafka            | - | - | 9092 | Message broker |
| Zookeeper        | - | - | 2181 | Coordenação Kafka |
| Redis            | - | - | 6379 | Cache   |

### Payload do Endpoint /alerts

```json
{
  "alertType": "SECURITY", // OBRIGATÓRIO
  "clientId": "´123", // OBRIGATÓRIO
  "message": "Intrusão detectada no setor 7",  // OBRIGATÓRIO
  "severity": "MEDIUM", // OBRIGATÓRIO
  "source": "Camera-01"
}
```
O campo severity aceita os valores: `LOW`, `MEDIUM`, `HIGH` , `CRITICAL`.

### 1️⃣ Enviar um Alerta

Coleção no Postman: [Link para Postman](https://www.postman.com/altimetry-engineer-56943415/desafio-ubisafe/collection/g61oww7/desafio-ubisafe?action=share&creator=41177636)

Ou use o comando curl abaixo:

```bash
curl -X POST http://localhost:8080/alerts \
  -H "Content-Type: application/json" \
  -d '{
    "alertType": "EMAIL",
    "clientId": "123",
    "message": "Intrusão detectada no setor 5",
    "severity": "MEDIUM",
    "source": "Camera-01"
}'
```

**Resposta esperada (202 Accepted):**
```json
{
  "message": "Alert received and queued for processing",
  "id": "983d554e-9279-4490-9c63-65ebf40f6776",
  "status": "ACCEPTED"
}
```

**Exemplo de resposta de erro (500 Internal Server Error):**
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred",
  "timestamp": "2025-11-17T12:34:56.789Z",
  "status": 500
}
```

### 2️⃣ Verificar Processamento

```bash
# Ver logs do processador
docker-compose logs -f alert-processor

# Você verá logs como:
# alert-processor | Processing alert for clientId: 12345
# alert-processor | Alert processed successfully with status: PROCESSADO
```

### 3️⃣ Verificar no Banco de Dados

```bash
# Conectar ao MySQL
docker exec -it mysql mysql -u root -proot alerts_db

# Consultar os alertas
SELECT * FROM alerts ORDER BY id DESC LIMIT 10;

# Sair do MySQL
exit;
```

##  Comandos Úteis
### Monitoramento

```bash
# Ver uso de recursos
docker stats

# Ver processos rodando em um container
docker top notification-api

# Inspecionar um container
docker inspect notification-api

# Ver rede
docker network inspect infra-notification-system_ubisafe-network
```




##  Decisões de Arquitetura


O sistema é composto por dois microsserviços independentes que se comunicam de forma assíncrona através do Apache Kafka:

```
Cliente → [notification-api] → Kafka (alerts topic) → [alert-processor] → MySQL
                ↓                                              ↓
              Redis                                     Registro de Falhas
         (Deduplicação)
```

### Características Principais

- Processamento assíncrono de alertas
- Deduplicação automática (janela configurável)
- Garantia de entrega com Kafka
- Persistência transacional com tratamento robusto de falhas
- Rastreabilidade completa de sucesso e erros

---

## Microsserviços

### notification-api (Produtor)

**Responsabilidades:**
- Receber requisições HTTP de criação de alertas
- Validar payload de entrada (Bean Validation)
- Verificar duplicação usando Redis
- Publicar mensagem no tópico Kafka `alerts`
- Responder imediatamente com HTTP 202 (Accepted)

### alert-processor (Consumidor)

**Responsabilidades:**
- Consumir mensagens do tópico `alerts`
- Deserializar e validar alertas
- Processar com delay simulado (500ms)
- Persistir alertas processados no MySQL
- Registrar falhas em transação independente

---



### 1. Comunicação Assíncrona com Kafka

**Por quê?**
- ✅ **Desacoplamento**: API e Processador não conhecem um ao outro
- ✅ **Resiliência**: Se o processador cair, mensagens ficam no Kafka
- ✅ **Escalabilidade**: Possível adicionar múltiplas instâncias do processador
- ✅ **Performance**: API responde imediatamente (202) sem aguardar processamento
- ✅ **Garantia de entrega**: Kafka garante que mensagens não sejam perdidas

**Configuração do Producer:**
```java
configProps.put(ProducerConfig.ACKS_CONFIG, "all");
configProps.put(ProducerConfig.RETRIES_CONFIG, 3);
configProps.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 1);
```

- `acks=all`: Espera confirmação de todos os brokers in-sync
- `retries=3`: Tenta reenviar até 3 vezes em caso de falha
- `max.in.flight=1`: Garante ordem das mensagens

**Configuração do Consumer:**
```java
configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.RECORD);
```

- Commit manual de offset somente após processamento completo
- Mensagens com erro podem ser reprocessadas ou enviadas para DLT

---

### 2. Deduplicação com Redis

**Implementação:**
```java
public boolean isDuplicate(String alertId) {
    String key = PREFIX + alertId;
    Boolean firstTime = redisTemplate.opsForValue()
        .setIfAbsent(key, "1", windowSeconds, TimeUnit.SECONDS);
    return firstTime == null || !firstTime;
}
```

**Por quê?**
- ✅ Evita processamento duplicado de alertas idênticos em curto período
- ✅ Redis in-memory é extremamente rápido
- ✅ TTL automático: chaves expiram após janela configurável
- ✅ Operação SETNX (Set If Not Exists) é atômica

**Critério de Duplicação:**
- Mesma combinação de `clientId + alertType + message + severity`
- `timestamp` e `source` não são considerados
- Hash determinístico garante mesmo ID para alertas idênticos

---

### 3. Pattern Produtor-Consumidor

**notification-api (Produtor):**
- Responsabilidade única: validar e publicar
- Não conhece quem vai processar
- Responde rapidamente ao cliente

**alert-processor (Consumidor):**
- Responsabilidade única: processar e persistir
- Não conhece quem enviou
- Processa no seu próprio ritmo

---

### 4. Delay Simulado (500ms)

**Implementação:**
```java
private static final long PROCESSING_DELAY_MS = 500;

Thread.sleep(PROCESSING_DELAY_MS);
```

**Justificativa:**
- Simula processamento real (envio de email, validações externas, etc.)
- Demonstra o benefício do processamento assíncrono
- Cliente recebe 202 imediatamente, sem esperar os 500ms
- Facilita visualização do fluxo em demonstrações

---

### 5. Persistência Transacional

#### Camada de Infraestrutura (Listener Kafka)

O listener consome a mensagem e delega para o serviço de aplicação. Não deve carregar a responsabilidade de transação nem fazer lógica de negócio.

```java
@KafkaListener(topics = "alerts", groupId = "processor-group")
public void consumeAlert(String alertJson) {
    // Sem @Transactional – apenas orquestra o fluxo
    alertService.processAlert(alertJson);
}
```

#### Camada de Aplicação (Serviços)

Responsável por processar, mapear e persistir. Separa fluxo principal e gravação de falhas em serviços distintos.

**Fluxo de Sucesso – Transação Única:**
```java
@Transactional
public AlertEntity processAlert(String alertJson) {
    Alert alert = objectMapper.readValue(alertJson, Alert.class);
    AlertEntity entity = alertMapper.toSuccessEntity(alert);
    return alertRepository.save(entity);
}
```

**Fluxo de Falha – Transação Independente:**
```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void registerFailureFromAlertJson(String alertJson, String failureReason) {
    Alert alert = objectMapper.readValue(alertJson, Alert.class);
    AlertEntity failedEntity = alertMapper.toFailureEntityFromAlert(alert, failureReason);
    alertRepository.save(failedEntity);
}
```

---

### 6. Tratamento de Exceções

**Hierarquia:**
```
AlertProcessingException (base)
    └── InvalidAlertJsonException (JSON malformado)
```

**Estratégia:**
```java
try {
    alertService.processAlert(alertJson);
} catch (InvalidAlertJsonException e) {
    // JSON inválido → salva payload bruto
    alertFailureService.registerFailureFromRawPayload(alertJson, reason);
    throw e;
} catch (AlertProcessingException e) {
    // Erro de negócio/técnico → salva dados do alerta
    alertFailureService.registerFailureFromAlertJson(alertJson, reason);
    throw e;
}
```

---

### 7. Mapper Dedicado

```java
@Component
public class AlertMapper {
    public AlertEntity toSuccessEntity(Alert alert) { ... }
    public AlertEntity toFailureEntityFromAlert(Alert alert, String reason) { ... }
    public AlertEntity toFailureEntityFromRawPayload(String payload, String reason) { ... }
}
```

**Por quê?**
- Evita duplicação de código
- Facilita manutenção e evolução
- Separação clara de responsabilidades
- Testabilidade isolada

---

## Fluxos Transacionais

### Fluxo de Sucesso

1. **Listener Kafka** recebe a mensagem
2. `alertService.processAlert()` inicia a transação **T1**
3. Desserializa JSON → mapeia entidade → persiste no MySQL
4. **Se tudo ocorreu bem:**
  - Commit de **T1**
  - Kafka confirma o offset
5. **Se ocorrer falha:**
  - Rollback automático de **T1**
  - Offset não confirmado → mensagem será reprocessada

### Fluxo de Falha

1. Ocorre erro no listener ou no serviço (JSON inválido, falha no MySQL etc.)
2. `alertFailureService.registerFailure*()` abre nova transação **T2** (`REQUIRES_NEW`)
3. Log de falha gravado no MySQL
4. Commit de **T2**, independente de **T1**
5. Exceção relançada → rollback de **T1**
6. Offset não confirmado → mensagem será reprocessada ou enviada para DLT

---

## Benefícios da Arquitetura

- ✅ **Integridade de dados:** ou processa com sucesso ou registra falha separadamente
- ✅ **Transações independentes:** rollback do fluxo principal não remove logs de falha
- ✅ **Separação de responsabilidades:** listener só orquestra; serviços fazem o trabalho pesado
- ✅ **Resiliência:** `REQUIRES_NEW` garante registro de falhas mesmo com erros no fluxo principal
- ✅ **Rastreabilidade:** falhas ficam armazenadas com timestamp e motivo detalhado
- ✅ **Escalabilidade:** Kafka distribui carga entre consumers; múltiplas instâncias possíveis
- ✅ **Testabilidade:** serviços desacoplados e injetáveis; mapper testável isoladamente

---

## Princípios Aplicados

| Princípio | Aplicação |
|-----------|-----------|
| **Single Responsibility** | Cada componente tem uma responsabilidade clara |
| **Open/Closed** | Fácil extensão sem modificar código existente |
| **Dependency Inversion** | Dependências via interfaces (Repository, ObjectMapper) |
| **Separation of Concerns** | Camadas bem definidas (infra, aplicação, domínio) |
| **Fail-Fast** | Validações no início do fluxo |
| **Defensive Programming** | Tratamento robusto de exceções |



### 8. Status do Alerta

Cada alerta processado tem um status final:

- **`SUCCESS`**: Processamento bem-sucedido
- **`FAILURE`**: Erro durante o processamento

### 9. Uso de Imagens Docker

**Estratégia:**
- Imagens dos microsserviços publicadas no Docker Hub
- Facilita deployment e distribuição
- Não precisa fazer build localmente
- Download automático das imagens

### 10. Separação em 3 Repositórios

**Benefícios:**
- Cada serviço evolui independentemente
- CI/CD isolado por serviço
- Facilita manutenção e versionamento
- Repositório umbrella como ponto único de entrada

### Imagens Docker

- `dyelll/notification-api:latest` - [Docker Hub](https://hub.docker.com/r/dyelll/notification-api)
- `dyelll/alert-processor:latest` - [Docker Hub](https://hub.docker.com/r/dyelll/alert-processor)


## 👥 Autor

Desenvolvido para o Desafio Ubisafe - Sistema de Notificação Assíncrona

Dylan Bitencourt Gonçalves

---

**Status:** ✅ Pronto para uso

**Última atualização:** Novembro 2025
