# Kafka with KafbatUI Guide

This guide provides a comprehensive walkthrough for setting up and interacting with a Kafka environment using **Docker Compose**, **KafbatUI**, and **Confluent's CLI tools**.  

It covers everything from the initial setup to producing and consuming messages, including a practical demonstration of **Schema Registry** with **Avro messages**.

---

## 📑 Table of Contents

- [Introduction](#kafka-with-kafbatui-guide)  
- [Prerequisites](#prerequisites)  
- [Installation](#installation)  
- [Environment](#environment)  
- [Client-1: Basic Console Producer/Consumer](#️client-1-basic-console-producerconsumer)  
- [Client-2: Avro Producer/Consumer with Schema Registry](#client-2-avro-producerconsumer-with-schema-registry)  
- [Client-3: Java Clients](#client-3-java-clients)  
- [Client-4: REST API](#client-4-rest-api)  
- [Clean-up](#clean-up)  

---

## Prerequisites

- 🐳 Running **Docker**  
- 📦 **Gradle**  
- ☕ **Java 11 or 17**  
- 🔧 **Maven** (required for Client-3: Java Clients)  
- ⚠️ Run this installation in advance as it involves some large downloads  

📖 Reference: [Confluent Prerequisites](https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html#prerequisites)

---

## Installation

1. **Clone the Repository**
   ```bash
   git clone https://github.com/namdeoankush/kafka-with-kafbatUI.git
   ```

2. **Navigate to the Root Directory**
   ```bash
   cd kafka-with-kafbatUI
   ```

3. **Start the Docker Containers**  
   > ⏳ The initial run may take a few minutes as Docker pulls the required images.  
   ```bash
   docker compose up -d
   ```

---

## Environment

Before we begin, let’s explore the tools available to manage our Kafka environment:

1. **Access the User Interfaces**
   - Kafka Control Center → `http://localhost:9021`  
   - KafbatUI → `http://localhost:8080`  

2. **Explore the UI Tabs**  
   Check **Topics**, **Schema Registry**, and **Consumers** tabs to monitor your Kafka cluster.  

3. **Check Container Logs**  
   Use Docker Desktop or CLI to view real-time logs for any container.

---

## Client-1: Basic Console Producer/Consumer

Kafka CLI tools are pre-installed inside the `broker` container.

### 1. Create a Topic (Optional)
```bash
docker-compose exec broker kafka-topics --create --topic data   --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

### 2. Verify Topic Creation
```bash
docker-compose exec broker kafka-topics --list --bootstrap-server broker:9092
```

### 3. Create a Console Producer
```bash
docker-compose exec broker kafka-console-producer   --bootstrap-server broker:9092 --topic data
```

### 4. Create a Console Consumer

👉 Always run the **consumer** in a separate terminal window so you can observe messages as they are produced.

```bash
docker-compose exec broker kafka-console-consumer   --bootstrap-server broker:9092 --topic data
```

👉 To read messages from the beginning:
```bash
docker-compose exec broker kafka-console-consumer   --bootstrap-server broker:9092 --topic data --from-beginning
```

💡 Stop consumer with: `Ctrl + C` (Linux/Windows) or `Cmd + C` (Mac).

---

## Client-2: Avro Producer/Consumer with Schema Registry

This section demonstrates producing and consuming Avro messages with **Schema Registry**.

### 1. Create `users` Topic
```bash
docker-compose exec broker kafka-topics --create --topic users   --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

### 2. Verify Topic
```bash
docker-compose exec broker kafka-topics --list --bootstrap-server broker:9092 | grep users
```

### 3. Create a Schema File
Create `user-schema.json` with:
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
```bash
docker cp user-schema.json schema-registry:/tmp/user-schema.json
```

### 5. Start Avro Console Producer
```bash
docker-compose exec schema-registry kafka-avro-console-producer   --bootstrap-server broker:29092   --topic users   --property schema.registry.url=http://schema-registry:8081   --property value.schema.file=/tmp/user-schema.json
```

👉 Example messages:
```json
{"name":"Charlie","age":28}
{"name":"Dana","age":35}
```

### 6. Validate Schema
- ✅ Valid: `{"name":"Dana","age":35, "test":1}`  
- ❌ Invalid: `{"name":"Dana","test":1}`  

### 7. Start Avro Console Consumer

👉 Always run the **consumer** in a separate terminal window so you can observe messages as they are produced.

```bash
docker-compose exec schema-registry kafka-avro-console-consumer   --bootstrap-server broker:29092   --topic users --from-beginning   --property schema.registry.url=http://schema-registry:8081
```

---

## Client-3: Java Clients

### 1. Basic Java Producer/Consumer
Use [Confluent Tutorials](https://github.com/confluentinc/tutorials/) with **Gradle + Java**.

### 2. Avro Java Producer/Consumer

- **Maven** → [Confluent Schema Registry Tutorial](https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html)  
- **Gradle** → [Kafka Schema Registry Clients Repo](https://github.com/namdeoankush/kafka-schema-registry-clients.git)

<details>
<summary>📂 Steps for Gradle Client (click to expand)</summary>

#### Clone and Build
```bash
git clone https://github.com/namdeoankush/kafka-schema-registry-clients.git
cd kafka-schema-registry-clients
gradle wrapper
./gradlew clean build
```

#### Produce & Consume Avro Messages

👉 Always run the **consumer** in a separate terminal window so you can observe messages as they are produced.

```bash
./gradlew run --args="avro produce"
./gradlew run --args="avro consume"
```

<details>
<summary>🔄 Schema Evolution Walkthrough (click to expand)</summary>

### 1. Delete a Field
Delete the `name` field from the schema.  
Update the Avro producer code so it does not populate this field.  
> ⚠️ If you don’t remove it from the code, the build will fail with a compilation error.

Rebuild the application:
```bash
./gradlew clean build
```

Run the same produce command:
```bash
./gradlew run --args="avro produce"
```

✅ This works because deleting a field is **backward compatible**.  
Now check the UI — you should see **Schema Version 2** registered.

---

### 2. Add a New Field
Now, try to add:
```json
{"name": "address", "type": "string"}
```
to the schema.  

Rebuild and produce again:
```bash
./gradlew clean build
./gradlew run --args="avro produce"
```

❌ This fails, because adding a required field without a default value is **not backward compatible**.

</details>

</details>

---

## Client-4: REST API

Follow on-screen instructions via **Confluent REST Proxy** (if enabled).

---

## Clean-up

To remove containers and volumes:

```bash
docker-compose down -v
```

---
