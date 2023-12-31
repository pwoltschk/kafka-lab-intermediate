# Kafka Cluster Lab

Welcome to the Kafka Cluster Lab! This practical exercise aims to 
provide a hands-on understanding of Apache Kafka's functionalities by 
setting up and exploring a Kafka cluster environment.

### Starting the Cluster
Before starting make sure you setup the environtment correctly see `~/setup.md`
Within the `~/kafka-cluster` folder execute `docker compose start`. You can stop the environment later with `docker compose stop`.

#### Starting 3 Zookeeper Hosts

```bash
# Zookeeper 1
docker run -d \
   --net=host \
   --name=zk-1 \
   -e ZOOKEEPER_SERVER_ID=1 \
   # ... [other environment variables] ...

# Zookeeper 2
docker run -d \
   --net=host \
   --name=zk-2 \
   -e ZOOKEEPER_SERVER_ID=2 \
   # ... [other environment variables] ...

# Zookeeper 3
docker run -d \
   --net=host \
   --name=zk-3 \
   -e ZOOKEEPER_SERVER_ID=3 \
   # ... [other environment variables] ...
```

#### Starting 3 Broker Instances

```bash
# Broker 1
docker run -d \
    --net=host \
    --name=kafka-1 \
    -e KAFKA_BROKER_ID=1 \
    # ... [other environment variables] ...

# Broker 2
docker run -d \
    --net=host \
    --name=kafka-2 \
    -e KAFKA_BROKER_ID=2 \
    # ... [other environment variables] ...

# Broker 3
docker run -d \
     --net=host \
     --name=kafka-3 \
     -e KAFKA_BROKER_ID=3 \
     # ... [other environment variables] ...
```

### Exercise 1: Cluster and Topics, Produce and Consume Messages

Let's dive into interacting with the cluster and understand message production and consumption.

#### Starting an Interactive Shell

```bash
docker run -it --net=host --rm  confluentinc/cp-kafka:7.3.2 /bin/bash
cd /bin
```

#### Create a Sensor Topic

```bash
./kafka-topics --bootstrap-server localhost:19092 --create --topic sensor --partitions 1 --replication-factor 3 --if-not-exists
```

After creating the topic, explore the following:

- Identify the leader broker.
- Shutdown the leader and observe the new leader.
- Start the broker that was previously offline.

#### Generating Sensor Data

```bash
seq 42 | ./kafka-console-producer --broker-list localhost:19092,localhost:29092,localhost:39092 --topic sensor && echo 'Produced 42 messages.'
```

#### Check Sensor Data

```bash
./kafka-console-consumer --bootstrap-server localhost:19092,localhost:29092,localhost:39092 --from-beginning --topic sensor
```

Explore behavior on:

- Shutdown of a single broker.
- Shutdown of two brokers.
- Replication behavior when producing data with only one broker active.

#### Failover Experiment

- Create a stream (`seq 10901101042 |...`).
- Connect a consumer.
- Observe leader changes by starting and stopping brokers.

### Exercise 2: Partitioning, Partitioning Keys, Consumer Groups

#### Create Sensor2 Topic

```bash
./kafka-topics --bootstrap-server localhost:19092 --create --topic sensor2 --partitions 2 --replication-factor 3 --if-not-exists
```

#### Generate Sensor2 Data

```bash
seq 42 | sed 's/\([0-9]\+\)/\1:\1/g' | ./kafka-console-producer --broker-list localhost:19092 --topic sensor2  --property parse.key=true --property key.separator=: && echo 'Produced 42 messages.'
```

#### Read Sensor2 Data

```bash
./kafka-console-consumer --bootstrap-server localhost:19092,localhost:29092,localhost:39092 --from-beginning --topic sensor2
```

Explore behavior with:

- Reading data with a second consumer in parallel.
- Reading data with two consumers from the same consumer group.
- Observing changes when new data is produced.

### Exercise 3: Log Compaction

This exercise focuses on configuring the `sensor3` topic for log compaction.

#### Configure Sensor3 Topic

```bash
# Topic creation with specific configurations
kafka-topics --bootstrap-server localhost:19092 --create --topic sensor3 --partitions 1 --replication-factor 3 --if-not-exists

# Setting log compaction parameters
kafka-configs --alter --add-config cleanup.policy=compact --entity-type topics --entity-name sensor3 --bootstrap-server localhost:19092
# ... [other parameters setup]
```

#### Produce and Observe Data Behavior

```bash
# Produce data with keys and observe behavior
seq 5 | sed 's/\([0-9]\+\)/\1:\1/g' | ./kafka-console-producer --broker-list localhost:19092 --topic sensor3  --property parse.key=true --property key.separator=: && echo 'Produced 5 messages.'
```

```bash
# Read data from sensor3 topic
./kafka-console-consumer --bootstrap-server localhost:19092,localhost:29092,localhost:39092 --from-beginning --topic sensor3
```

Lab Exploration:

- Repeating data multiple times and observing behavior.
- Producing a sequence of entries, reading data, and then producing another sequence.
- Optional: Manually sending data and observing topic behavior.

Feel free to explore and experiment further to gain a deeper understanding of Kafka's behavior and functionalities within this lab environment.
