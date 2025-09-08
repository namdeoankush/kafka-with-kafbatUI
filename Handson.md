# To Do

## Table of Contents
- [Installation](#installation)
- [Navigating](#navigating)
- [Clients -basic](#clients-basic)
- [Clients -2](#clients -2)


## Installation
1. Clone this repository:
```bash
 git clone https://github.com/namdeoankush/kafka-with-kafbatUI.git
```
2. Go inside the root folder
```bash
 cd kafka-withkafbatUI
```

3. Run docker compose to bring to container (if this is the first time it can take few minutes as it will pull all the repository from docker hub, being in really good network)
```bash
 docker compose up -d
```

## Navigating

Before we move on to the actual data,I want us to see what are environment looks like.

1. Go into localhost:9021 to view control center or localhost:8080 for kafBatUI.
   
2. Look for different tabs, topic, schema registory, consumer.

3. To check logs on console you can use docker-desktop or your favourite docker tool.

4. kafka-UI 101 (creating topic, publishing a message, viewing topic and messages)

## Clients -1

Now we are going to see how we can use this enviornment for our need. 

1. Using console commands for producer and consumer (basic).

to use this we need to have kafka CLIs installed, for today We can use pre installed cli on your docker enviornment.

# optional

Create a topic using CLI 

```bash
docker-compose exec broker \
  kafka-topics --create \
  --topic data \
  --bootstrap-server broker:9092 \
  --partitions 1 \
  --replication-factor 1
```

```bash
docker-compose exec broker kafka-topics --create --topic data --bootstrap-server broker:9092 --partitions 1 --replication-factor 1
```

Verify if the topic is created. Go to kafbatUi or control center. Otherwise we can run list command to get the topic

```bash
docker-compose exec broker kafka-topics --list --bootstrap-server broker:9092
```


# Creating first producer 

```bash
docker-compose exec broker \
  kafka-console-producer \
  --broker-list broker:9092 \
  --topic data
```

```bash
docker-compose exec broker kafka-console-producer --bootstrap-server broker:9092 --topic my-topic
```

Verify the message on UI

# creating first consumer

```bash
docker-compose exec broker \
  kafka-console-consumer \
  --bootstrap-server broker:9092 \
  --topic data
```

```bash
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic data
```

Cant see any messages yet?
# missing : --from-beginning

```bash
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic data --from-beginning
```


# Deleting a topic (Optional)

```bash
docker-compose exec broker kafka-topics --delete --topic my-topic --bootstrap-server broker:9092
```

## Clients -2

Now we have seen the how we can produce and consume message, Now moving ahead, as we have talked about schema and using schema registry now We are going to see that in practical. The CP-server have basic kafka cli not the schema registry cli so we will do that against schema registry container. Alternatively we could install confluent cli, or cp-tools which gives access to these advance cli option out of box.


# Create a schema file first

we have some formatting issue on windows(I am not an expert so I thought of alternative approch)

notepad user-scheam1.json

paste below schema and save it: 
{"type":"record","name":"User","fields":[{"name":"name","type":"string"},{"name":"age","type":"int"}]}


# Then copy it to the container and reference it
```bash
docker cp user-schema.json schema-registry:/tmp/user-schema.json


docker-compose exec schema-registry kafka-avro-console-producer --bootstrap-server broker:9092 --topic users --property schema.registry.url=http://schema-registry:8081 --property value.schema.file=/tmp/user-schema.json

```
 past this message once your curser is idle, one by one
{"name":"Charlie","age":28}

{"name":"Dana","age":35}


verify the message on UI.

you can also see the schema on schema registry section.

Now if you have producer running already try  publishing a message which doesnt follow the same schema.
{"name":"Dana","age":35, "test":1}

now try another message

{"name":"Dana", "test":1}

this last message should give you error. This is one use case or one good thing about using schema registry. your data doesnt end up on kafka and is rejected from schema registry already before being published to kafka.


Now Lets consume this message from consumer, lets try runnning normal consumer on this topic.

```bash
docker-compose exec broker \
  kafka-console-consumer \
  --bootstrap-server broker:9092 \
  --topic users
```

```bash
docker-compose exec broker kafka-console-consumer --bootstrap-server broker:9092 --topic users --from-beginning
```

now try avro console consumer : 

```bash
docker-compose exec schema-registry kafka-avro-console-consumer --bootstrap-server broker:29092 --topic users --from-beginning --property schema.registry.url=http://schema-registry:8081
```

