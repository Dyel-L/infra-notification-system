# Sistema de Notificação Assíncrona - Ubisafe

Sistema distribuído de notificação assíncrona com arquitetura baseada em microsserviços, utilizando Java/Spring Boot, Apache Kafka e MySQL.

## 📋 Visão Geral

Este é o **repositório de infraestrutura** que orquestra todos os componentes do sistema através do Docker Compose. O sistema implementa um fluxo assíncrono de processamento de alertas composto por dois microsserviços independentes:

1. **notification-api** - Recebe alertas via REST API e os publica no Kafka
2. **alert-processor** - Consome alertas do Kafka, processa (com delay simulado de 500ms) e persiste no MySQL

### Arquitetura do Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                     Cliente (curl, Postman)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTP POST
                             ▼
                  ┌──────────────────────┐
                  │  notification-api    │
                  │    (porta 8080)      │
                  │  - Valida payload    │
                  │  - Retorna 202       │
                  └──────────┬───────────┘
                             │ Publica mensagem
                             ▼
                  ┌──────────────────────┐
                  │   Apache Kafka       │
                  │   Tópico: alerts     │
                  │  (Message Broker)    │
                  └──────────┬───────────┘
                             │ Consome mensagem
                             ▼
                  ┌──────────────────────┐
                  │  alert-processor     │
                  │  - Consome do Kafka  │
                  │  - Delay 500ms       │
                  │  - Persiste no DB    │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │      MySQL 8.0       │
                  │   alerts_db          │
                  │  (Persistência)      │
                  └──────────────────────┘
```

## 🏗️ Estrutura dos Repositórios

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

## 🚀 Tecnologias Utilizadas

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

### Testes
- **JUnit 5** - Framework de testes
- **Mockito** - Mocks para testes unitários
- **Spring Boot Test** - Testes de integração

## 🔧 Pré-requisitos

- **Docker 20.10+**
- **Docker Compose 2.0+**

## 🏃 Como Executar

### ⚡ Início 



```bash
# 1. Clone este repositório
git clone https://github.com/seu-usuario/infra-notification-system.git
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
2. Subir Zookeeper e Kafka
3. Subir MySQL e criar o banco `alerts_db`
4. Subir os dois microsserviços

### 📊 Verificar Status

```bash
# Ver status de todos os containers
docker-compose ps

# Resultado esperado:
NAME               IMAGE                             COMMAND                  SERVICE            CREATED         STATUS                   PORTS
alert-processor    dyelll/alert-processor:latest     "java -jar app.jar"      alert-processor    2 minutes ago   Up 2 minutes
kafka              confluentinc/cp-kafka:7.5.0       "/etc/confluent/dock…"   kafka              2 minutes ago   Up 2 minutes             0.0.0.0:9092->9092/tcp, [::]:9092->9092/tcp
mysql              mysql:8.0                         "docker-entrypoint.s…"   mysql              2 minutes ago   Up 2 minutes (healthy)   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp
notification-api   dyelll/notification-api:latest    "java -jar app.jar"      notification-api   2 minutes ago   Up 2 minutes             0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp
zookeeper          confluentinc/cp-zookeeper:7.5.0   "/etc/confluent/dock…"   zookeeper          2 minutes ago   Up 2 minutes             0.0.0.0:2181->2181/tcp, [::]:2181->2181/tcp
```

### 📝 Acompanhar Logs

```bash
# Ver logs de todos os serviços
docker-compose logs -f

# Ver logs de um serviço específico
docker-compose logs -f notification-api
docker-compose logs -f alert-processor

# Ver apenas as últimas 100 linhas
docker-compose logs --tail=100 -f
```

## 📡 Testando o Sistema

## 📊 Endpoints Disponíveis

| Serviço | Endpoint | Método | Porta | Descrição |
|---------|----------|--------|-------|-----------|
| notification-api | `/alerts` | POST | 8080 | Criar novo alerta |
| MySQL | - | - | 3306 | Banco de dados |
| Kafka | - | - | 9092 | Message broker |
| Zookeeper | - | - | 2181 | Coordenação Kafka |

### Payload do Endpoint /alerts

```json
{
  "alertType": "SECURITY", // OBRIGATÓRIO
  "clientId": "´123", // OBRIGATÓRIO
  "message": "Intrusão detectada no setor 7",  // OBRIGATÓRIO
  "severity": "MEDIUM",
  "source": "Camera-01"
}
```

### 1️⃣ Enviar um Alerta

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

## 🛠️ Comandos Úteis
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




## 🏗️ Decisões de Arquitetura

### 1. Comunicação Assíncrona com Kafka

**Por quê?**
- ✅ **Desacoplamento**: API e Processador não conhecem um ao outro
- ✅ **Resiliência**: Se o processador cair, mensagens ficam no Kafka
- ✅ **Escalabilidade**: Possível adicionar múltiplas instâncias do processador
- ✅ **Performance**: API responde imediatamente (202) sem aguardar processamento
- ✅ **Garantia de entrega**: Kafka garante que mensagens não sejam perdidas

### 2. Pattern Produtor-Consumidor

**notification-api (Produtor):**
- Responsabilidade única: validar e publicar
- Não conhece quem vai processar
- Responde rapidamente ao cliente

**alert-processor (Consumidor):**
- Responsabilidade única: processar e persistir
- Não conhece quem enviou
- Processa no seu próprio ritmo

### 3. Delay Simulado (500ms)

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

### 4. Persistência Transacional

**Configuração:**
O listener consome a mensagem e delega para o serviço de aplicação. Não deve carregar a responsabilidade de transação nem fazer lógica de negócio.

**Configuração:**
```java
@KafkaListener(topics = "alerts", groupId = "processor-group")
public void consumeAlert(String alertJson) {
    // Sem @Transactional – apenas orquestra o fluxo
    alertService.processAlert(alertJson);
}
```

## Camada de Aplicação (Serviços)

Responsável por processar, mapear e persistir. Separa fluxo principal e gravação de falhas em serviços distintos.

### Fluxo de Sucesso – Transação Única

**Configuração:**
```java
@Transactional
public AlertEntity processAlert(String alertJson) {
    Alert alert = objectMapper.readValue(alertJson, Alert.class);
    AlertEntity entity = alertMapper.toSuccessEntity(alert);
    return alertRepository.save(entity);
}
```

### Fluxo de Falha – Transação Independente

**Configuração:**
```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void registerFailureFromAlertJson(String alertJson, String failureReason) {
    Alert alert = objectMapper.readValue(alertJson, Alert.class);
    AlertEntity failedEntity = alertMapper.toFailureEntityFromAlert(alert, failureReason);
    alertRepository.save(failedEntity);
}
```
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
6. Offset não confirmado → mensagem será reprocessada ou enviada para DLT, dependendo da config

---

## Benefícios da Arquitetura

- ✅ **Integridade de dados:** ou processa com sucesso ou registra falha separadamente
- ✅ **Transações independentes:** rollback do fluxo principal não remove logs de falha
- ✅ **Separação de responsabilidades:** listener só orquestra; serviços fazem o trabalho pesado
- ✅ **Resiliência:** `REQUIRES_NEW` garante registro de falhas mesmo com erros no fluxo principal
- ✅ **Rastreabilidade:** falhas ficam armazenadas com timestamp e motivo detalhado


### 5. Status do Alerta

Cada alerta processado tem um status final:

- **`SUCCESS`**: Processamento bem-sucedido
- **`FAILURE`**: Erro durante o processamento

### 6. Healthchecks e Dependências

**MySQL com healthcheck:**
```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

**Benefício:**
- alert-processor só inicia quando MySQL está pronto
- Evita erros de conexão durante startup

### 7. Uso de Imagens Docker

**Estratégia:**
- Imagens dos microsserviços publicadas no Docker Hub
- Facilita deployment e distribuição
- Não precisa fazer build localmente
- Download automático das imagens

### 8. Separação em 3 Repositórios

**Benefícios:**
- Cada serviço evolui independentemente
- CI/CD isolado por serviço
- Facilita manutenção e versionamento
- Repositório umbrella como ponto único de entrada

### Imagens Docker

- `dyelll/notification-api:latest` - [Docker Hub](https://hub.docker.com/r/dyelll/notification-api)
- `dyelll/alert-processor:latest` - [Docker Hub](https://hub.docker.com/r/dyelll/alert-processor)


## 📄 Licença

Este projeto foi desenvolvido como parte do desafio técnico Ubisafe.

## 👥 Autor

Desenvolvido para o Desafio Ubisafe - Sistema de Notificação Assíncrona

Dylan Bitencourt Gonçalves

---

**Status:** ✅ Pronto para uso

**Última atualização:** Novembro 2025