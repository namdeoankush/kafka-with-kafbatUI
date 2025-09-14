### Introduction

This guide provides a comprehensive walkthrough for setting up and interacting with a Kafka environment using Docker Compose, KafbatUI, and Confluent's command-line tools. It covers essential operations from initial setup to producing and consuming messages, including a practical demonstration of using Schema Registry with Avro messages.

-----

### Table of Contents

- [Introduction](#introduction)
- [Table of Contents](#table-of-contents)
- [Prerequisite](#Prerequisite)
- [Installation](#installation)
- [Environment](#environment)
- [Client-1: Basic Console Producer/Consumer](#client-1-basic-console-producerconsumer)
- [Client-2: Avro Producer/Consumer with Schema Registry](#client-2-avro-producerconsumer-with-schema-registry)
- [Client-3: Java Clients](#client-3-java-clients)
- [Client-4: Rest API](#client-4-rest-api)

-----

### Prerequisite

1. Running Docker
2. Run this installation in advance as it involves some large downloads.
    For Clients:
3. Gradle
4. Java 11 or 17
5. Maven for Client-3: Java Clients (Avro java consumer & Producer) // As mentioned in [https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html#prerequisites](https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html#prerequisites)

-----

### Installation

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
Start a console producer to send messages to the `data` topic. Type your messages and press Enter to send them.

```bash
docker-compose exec broker kafka-console-producer --bootstrap-server broker:9092 --topic data
```

**Creating a Console Consumer**


Start a console consumer to read messages from the `data` topic. You won't see any messages until you run the command with the `--from-beginning` flag.
In a new terminal window, run the following: 
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
In a new terminal window, run the following: 
**1. Create users Topic**
Create a new topic named `users` which we gonna use to push data serialized with a schema.

```bash
docker-compose exec broker kafka-topics --create --topic data --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

**Verify topic is created**
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

We will use [https://github.com/confluentinc/tutorials/](https://github.com/confluentinc/tutorials/) for creating Java clients. Prerequisites for this are to have Gradle and Java set up on machines. 

**2. Avro java consumer & Producer**

* **For Maven:**
We will use [https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html](https://docs.confluent.io/platform/current/schema-registry/schema_registry_onprem_tutorial.html). Prerequisites for this are to have Maven and Java set up on the machines.

* **For Gradle:**
We can use [https://github.com/namdeoankush/kafka-schema-registry-clients.git](https://github.com/namdeoankush/kafka-schema-registry-clients.git). This has AVRO, Protobuf, and JSON clients. For this exercise, we will focus only on AVRO clients.

            First, let's check out this repository:
```bash
git checkout https://github.com/namdeoankush/kafka-schema-registry-clients.git && cd kafka.schema-registry-clients
```

Now let's build the Gradle app.

**1. We need to run the Gradle wrapper first

```bash
gradle wrapper
```
**2. Once the wrapper command runs without any errors, it's time to build the application.

```bash
./gradlew clean build
```
**3. Producing Avro message on 'example-avro-topic' topic:

#### Produce an Avro message
./gradlew run --args="avro produce"

#### Consume Avro messages
./gradlew run --args="avro consume"

Now to check the compatibility, we will delete the Name field from the schema, change the code in the Avro producer so the code doesn't populate this field (not doing this will throw a  compilation error during build).

Now we will run the build command again: 

```bash
./gradlew clean build
```
and run the same produce command that we did earlier. As you can see, it allows you to send the message as the deleting field is backward compatible.

Now go to UI and check the schema; there should be a new schema version 2 available. 

Now to test when backward compatibility doesn't work 

We will try to add a new field in the schema and produce a new message using that field. 

Add {"name": "address", "type": "string"} in the schema. Build the Gradle app.

run the command to produce the message again. You should see an error similar to this: 

```bash
Schema being registered is incompatible with an earlier schema for subject "example-avro-topic-value", details: [{errorType:'READER_FIELD_MISSING_DEFAULT_VALUE', description:'The field 'address' at path '/fields/2' in the new schema has no default value and is missing in the old schema', additionalInfo:'address'}, {oldSchemaVersion: 2}, {oldSchema: '{"type":"record","name":"User","namespace":"com.example.avro","fields":[{"name":"id","type":"long"},{"name":"email","type":["null",{"type":"string","avro.java.string":"String"}],"default":null}]}'}, {validateFields: 'false', compatibility: 'BACKWARD'}]; error code: 409
```

-----

### Client-4: Rest API
Follow on screen

-----
### Clean-up
To destroy the docker environment use 

```bash
docker-compose down -v
```

