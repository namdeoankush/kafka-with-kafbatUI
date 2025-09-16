# Kafka with KafbatUI Guide (Windows)

## This is an AI generated file, so it needs validation

This guide provides a comprehensive walkthrough for setting up and interacting with a Kafka environment using **Docker Compose**, **KafbatUI**, and **Confluent's CLI tools** on **Windows PowerShell**.

It covers everything from the initial setup to producing and consuming messages, including a practical demonstration of **Schema Registry** with **Avro messages**.

---

## 📍 Table of Contents

- [Introduction](#kafka-with-kafbatui-guide-windows)  
- [Prerequisites](#prerequisites)  
- [Installation](#installation)  
- [Environment](#environment)  
- [Client-1: Basic Console Producer/Consumer](#client-1-basic-console-producerconsumer)  
- [Client-2: Avro Producer/Consumer with Schema Registry](#client-2-avro-producerconsumer-with-schema-registry)  
- [Client-3: Java Clients](#client-3-java-clients)  
- [Client-4: REST API](#client-4-rest-api)  
- [Clean-up](#clean-up)  

---

## Prerequisites

- 🐳 Running **Docker Desktop for Windows**  
- 📦 **Gradle** (use `gradlew.bat` on Windows)  
- ☕ **Java 11 or 17**  
- 🔧 **Maven** (for Client-3: Java Clients)  
- ⚠️ Run this installation in advance as it involves some large downloads  

📖 Reference: [Confluent Prerequisites](https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html#prerequisites)

---

## Installation

1. **Clone the Repository**
```powershell
git clone https://github.com/namdeoankush/kafka-with-kafbatUI.git
```

2. **Navigate to the Root Directory**
```powershell
cd kafka-with-kafbatUI
```

3. **Start the Docker Containers**  
> ⏳ The initial run may take a few minutes as Docker pulls the required images.  
```powershell
docker-compose up -d
```

---

## Environment

1. **Access the User Interfaces**
   - Kafka Control Center → `http://localhost:9021`  
   - KafbatUI → `http://localhost:8080`  

2. **Explore the UI Tabs**  
   Check **Topics**, **Schema Registry**, and **Consumers** tabs to monitor your Kafka cluster.  

3. **Check Container Logs**
```powershell
docker-compose logs -f broker
```

---

## Client-1: Basic Console Producer/Consumer

Kafka CLI tools are pre-installed inside the `broker` container.

### 1. Create a Topic
```powershell
docker-compose exec broker kafka-topics --create --topic data --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

### 2. Verify Topic Creation
```powershell
docker-compose exec broker kafka-topics --list --bootstrap-server broker:9092
```

### 3. Create a Console Producer
```powershell
docker-compose exec broker kafka-console-producer --bootstrap-server broker:9092 --topic data
```

### 4. Create a Console Consumer
> Run the **consumer** in a separate PowerShell window.
```powershell
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic data
```

To read messages from the beginning:
```powershell
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic data --from-beginning
```

💡 Stop consumer with: `Ctrl + C`.

---

## Client-2: Avro Producer/Consumer with Schema Registry

### 1. Create `users` Topic
```powershell
docker-compose exec broker kafka-topics --create --topic users --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

### 2. Verify Topic
```powershell
docker-compose exec broker powershell -Command "kafka-topics --list --bootstrap-server broker:9092 | Select-String users"
```

### 3. Create a Schema File
Create `user-schema.json`:
```json
{
  "type": "record",
  "name": "User",
  "fields": [
    {"name":"name","type":"string"},
    {"name":"age","type":"int"}
  ]
}
```

### 4. Copy Schema to Container
```powershell
docker cp .\user-schema.json schema-registry:/tmp/user-schema.json
```

### 5. Start Avro Console Producer
```powershell
docker-compose exec schema-registry kafka-avro-console-producer --bootstrap-server broker:29092 --topic users --property schema.registry.url=http://schema-registry:8081 --property value.schema.file=/tmp/user-schema.json
```

Example messages:
```json
{"name":"Charlie","age":28}
{"name":"Dana","age":35}
```

### 6. Start Avro Console Consumer
> Run the **consumer** in a separate PowerShell window.
```powershell
docker-compose exec schema-registry kafka-avro-console-consumer --bootstrap-server broker:29092 --topic users --from-beginning --property schema.registry.url=http://schema-registry:8081
```

---

## Client-3: Java Clients (Windows)

### 1. Basic Java Producer/Consumer

**Gradle commands on Windows** use `gradlew.bat`:

**Producer**
```powershell
.\gradlew.bat :kafka-producer-application:kafka:shadowJar
java -jar kafka-producer-application\kafka\build\libs\kafka-producer-application-standalone.jar host:port kafka-producer-application\kafka\input.txt
```

**Consumer**
```powershell
.\gradlew.bat :kafka-consumer-application:kafka:shadowJar
java -jar kafka-consumer-application\kafka\build\libs\kafka-consumer-application-standalone.jar host:port consumer1
```

### 2. Avro Java Producer/Consumer

Clone and build the Gradle Avro project:
```powershell
git clone https://github.com/namdeoankush/kafka-schema-registry-clients.git
cd kafka-schema-registry-clients
gradlew.bat clean build
```

**Produce & Consume**
```powershell
gradlew.bat run --args="avro produce"
gradlew.bat run --args="avro consume"
```

**Schema Evolution**
- Delete or add fields in schema as in the Mac version.
- Rebuild with `gradlew.bat clean build`.
- Produce/consume with the same commands.

---

## Client-4: REST API

Use **Confluent REST Proxy** as per on-screen instructions.  
PowerShell commands are identical to Mac, except use `.bat` scripts for Gradle if required.

---

## Clean-up

Remove containers and volumes:
```powershell
docker-compose down -v
```

---

✅ All commands are now **Windows PowerShell-compatible**, with path adjustments, `gradlew.bat`, and PowerShell-friendly piping (`Select-String`) for topic filtering.

