# Kafka Notes

My notes for getting started with [Kafka](https://kafka.apache.org/) on
a Mac. This is for development and testing. After experimenting with a
few other options [Confluent](https://www.confluent.io/) / [Confluent
Open Source](https://www.confluent.io/product/confluent-open-source/)
seems to provide the best setup for my taste.

### Architecture

```plain
     Stream Processors
           \ /
 Producers <-> Consumers
           / \
        Connectors
```

### Setup

Run [Zookeeper](https://zookeeper.apache.org/) , Kafka and
[Schema-registry](https://docs.confluent.io/current/schema-registry/docs/index.html) as seperate Docker containers:

```bash
docker run -d --name zookeeper \
    -p 2181:2181 \
    confluent/zookeeper
    
docker run -d --name kafka \
    -p 9092:9092 \
    --link zookeeper:zookeeper \
    confluent/kafka
    
docker run -d --name schema-registry \
    -p 8081:8081 --link zookeeper:zookeeper \
    --link kafka:kafka \
    confluent/schema-registry
```

Open a shell into the running Kafka container

`docker exec -it kafka`

From inside the new Kafka container make a **test** topic:

```bash
/usr/bin/kafka-topics --create \
    --zookeeper zookeeper:2181 \
    --replication-factor 1 \
    --partitions 1 \
    --topic test
```

List available topics:

```bash
/usr/bin/kafka-topics --list \
    --zookeeper zookeeper:2181
```

### Test

Open two more terminals on your Mac. One will be used for a consumer and one will be used for a publisher

##### Kafka Consumer

Listen for messages on the new **test** topic.

```bash
docker exec -it kafka bash

/usr/bin/kafka-console-consumer --zookeeper zookeeper:2181 \
    --topic test

```

##### Kafka Producer

Produce messages for the test topic in a seperate terminal.

```bash
docker exec -it kafka bash

/usr/bin/kafka-console-producer --broker-list localhost:9092 \
    --topic test
```

## Docker Compose

The following is Docker compose version of this setup:

```yaml
version: "3.2"

networks:
  kafkanet:
    driver: bridge

services:
  zookeeper:
    image: "confluent/zookeeper"
    container_name: "zookeeper"
    networks:
      - kafkanet
    ports:
      - 2881:2181

  kafka:
    image: "confluent/kafka"
    container_name: "kafka"
    depends_on:
      - zookeeper
    networks:
      - kafkanet
    ports:
      - 9092:9092

  schema-registry:
    image: "confluent/schema-registry"
    container_name: "schema-registry"
    depends_on:
      - kafka
    networks:
      - kafkanet
    ports:
      - 8081:8081
         
```

This Kafka notes repository contains this compose file.

```bash
git clone git@github.com:cjimti/notes-kafka.git

cd notes-kafka

docker-compose up
```
