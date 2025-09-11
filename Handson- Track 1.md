### Introduction

This guide provides a comprehensive walkthrough for setting up and interacting with a Kafka environment using Docker Compose, KafbatUI, and Confluent's command-line tools. It covers essential operations from initial setup to producing and consuming messages, including a practical demonstration of using Schema Registry with Avro messages.

-----

### Table of Contents

- [Introduction](#introduction)
- [Table of Contents](#table-of-contents)
- [Installation](#installation)
- [Environment](#environment)
- [Client-1: Basic Console Producer/Consumer](#client-1-basic-console-producerconsumer)
- [Client-2: Avro Producer/Consumer with Schema Registry](#client-2-avro-producerconsumer-with-schema-registry)
- [Client-3: Java Clients](#client-3-java-clients)
- [Client-4: Rest API](#client-4-rest-api)

-----

### Installation

**Prerequisite**

1. Running Docker
2. Run this installation in advance as it involves some large downloads.
    For Clients:
3. Gradle
4. Java 11 or 17
5. Maven for Client-3: Java Clients (Avro java consumer & Producer) // As mentioned in [https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html#prerequisites](https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html#prerequisites)


1.  **Clone the Repository**
    ```bash
    git clone https://github.com/namdeoankush/kafka-with-kafbatUI.git
    ```
2.  **Navigate to the Root Directory**
    ```bash
    cd kafka-with-kafbatUI
    ```
3.  **Start the Docker Containers**
    This command brings up all necessary services. The initial run may take a few minutes as Docker pulls the required images.
    ```bash
    docker compose up -d
    ```

-----

### Environment

Before we begin, let's explore the tools provided to manage our Kafka environment.

1.  **Access the User Interfaces**
      * **Kafka Control Center:** `http://localhost:9021`
      * **KafbatUI:** `http://localhost:8080`
2.  **Explore the UI Tabs**
    Familiarize yourself with the various tabs like **Topics**, **Schema Registry**, and **Consumers** to monitor your Kafka cluster.
3.  **Check Container Logs**
    You can view real-time logs for any container using Docker Desktop or your preferred Docker management tool.

-----

### Client-1: Basic Console Producer/Consumer

This section demonstrates how to use the basic Kafka command-line interface (CLI) tools, which are pre-installed within the `broker` container.

**Creating a Topic (Optional)**
You can create a new topic named `data` using the `kafka-topics` command.

```bash
docker-compose exec broker kafka-topics --create --topic data --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

**Verify the Topic Creation**
Confirm the topic's existence by checking KafbatUI/Control Center or by listing all topics from the command line.

```bash
docker-compose exec broker kafka-topics --list --bootstrap-server broker:9092
```

**Creating a Console Producer**
Start a console producer to send messages to the `data` topic. Type your messages and press enter to send them.

```bash
docker-compose exec broker kafka-console-producer --bootstrap-server broker:9092 --topic data
```

**Creating a Console Consumer**


Start a console consumer to read messages from the `data` topic. You won't see any messages until you run the command with the `--from-beginning` flag.
In a new terminal window run follwing: 
```bash
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic data
```
control +c or cmd + c to kill the consumer.

To view all messages from the beginning of the topic:

```bash
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic data --from-beginning
```

-----

### Client-2: Avro Producer/Consumer with Schema Registry

This section focuses on producing and consuming Avro messages, leveraging the power of Schema Registry for data validation.
In a new terminal window run follwing: 
**1. Create users Topic**
Create a new topic names `users` which we gonna use push data serialized with schema.

```bash
docker-compose exec broker kafka-topics --create --topic data --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

**Veriy topic is created**
Confirm the topic's existence by checking KafbatUI/Control Center or by listing all topics from the command line.

```bash
docker-compose exec broker kafka-topics --list --bootstrap-server broker:9092 |grep users
```

**2. Create a Schema File**
Create a new file named `user-schema.json` inside kafka-with-kafbatUI directory
 and paste the following JSON content into it.

```json
{"type":"record","name":"User","fields":[{"name":"name","type":"string"},{"name":"age","type":"int"}]}
```

**3. Copy the Schema to the Container**
Copy your local schema file into the `schema-registry` container so the producer can access it.

```bash
docker cp user-schema.json schema-registry:/tmp/user-schema.json
```

**4. AVRO Console Producer**
Run the Avro producer, referencing the schema file you just copied. The producer will validate messages against this schema before sending them to the `users` topic.

```bash
docker-compose exec schema-registry kafka-avro-console-producer --bootstrap-server broker:29092 --topic users --property schema.registry.url=http://schema-registry:8081 --property value.schema.file=/tmp/user-schema.json
```

Enter the following messages one by one:

  * `{"name":"Charlie","age":28}`
  * `{"name":"Dana","age":35}`

**Verify the Messages and Schema**
Check KafbatUI to see the published messages. You can also inspect the registered schema in the **Schema Registry** section of the UI.

**Test Schema Validation**
Try to publish a message that violates the schema. The Schema Registry will reject it, demonstrating its role in maintaining data integrity.

  * **Valid:** `{"name":"Dana","age":35, "test":1}`
  * **Invalid:** `{"name":"Dana", "test":1}` (This message should be rejected and not end up on the Kafka topic).

**4. AVRO Console Consumer**
You cannot read Avro messages with a standard console consumer because they are serialized with Avro formatting.
In a new terminal window run follwing: 

```bash
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic users --from-beginning
```

This will display unreadable output.

Instead, use the `kafka-avro-console-consumer` to deserialize the messages correctly.

```bash
docker-compose exec schema-registry kafka-avro-console-consumer --bootstrap-server broker:29092 --topic users --from-beginning --property schema.registry.url=http://schema-registry:8081
```

### Client-3: Java Clients

**1. Basic Java consumers & Producer**

We will use [https://github.com/confluentinc/tutorials/](https://github.com/confluentinc/tutorials/) for creating java clients. Prerequisites for this is to have Gradle, java setup on machines. 

**2. Avro java consumer & Producer**

We will use [https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html](https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html). Prerequisites for this is to have Maven, java setup on machines.

### Client-4: Rest API
Follow on screen


# Clean-up
To destroy the docker environment use 

```bash
docker-compose down -v
```

