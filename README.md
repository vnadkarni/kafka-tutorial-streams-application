# How to build your first Apache Kafka Streams application

***Question:***

*How do you get started building your first Kafka Streams application?*

Steps are from: **[How to build your first Apache Kafka Streams application](https://developer.confluent.io/tutorials/creating-first-apache-kafka-streams-application/confluent.html)**

The steps below are in a slightly different order from the Tutorial on the Confluent Developer website, so there are numbers in black circles (❶, ❷, etc) that refer to the step number in that Kafka tutorial at Confluent Developer.

---



## Provision the cluster
❶
1. Sign in to Confluent Cloud.
2. Create a cluster. Give it a descriptive name.



## Download and set up the Confluent CLI
❹
Install confluent CLI from here: [Install Confluent CLI](https://docs.confluent.io/confluent-cli/current/install.html#scripted-installation).
Install to `./bin`. Don't forget to add it to the `PATH` variable. Then use the following commands to login, allow auth using api-key and secret, explore environments and clusters:

`$ confluent login`

`$ confluent environment list`

`$ confluent environment use <env-id>`

`$ confluent api-key store --resource <cluster id>`

`$ confluent api-key store <key> <secret> --resource <cluster id>`

`$ confluent kafka cluster list`

`$ confluent kafka cluster use <cluster id>`



## Create a topic
 ❺ Create a topic in the Kafka cluster:
```
confluent kafka topic create output-topic --partitions 1
```



## Initialise the project
❷

Make the local directory for the project
```
mkdir kafka-producer-application && cd kafka-producer-application
```
Make a directory for configuration data:
```
mkdir configuration
```
❸ Write config info in files
`configuration/ccloud.properties` :

```

# Required connection configs for Kafka producer, consumer, and admin
bootstrap.servers={{ BOOTSTRAP_SERVERS }}
security.protocol=SASL_SSL
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username='{{ CLUSTER_API_KEY }}' password='{{ CLUSTER_API_SECRET }}';
sasl.mechanism=PLAIN
# Required for correctness in Apache Kafka clients prior to 2.6
client.dns.lookup=use_all_dns_ips

# Best practice for Kafka producer to prevent data loss
acks=all

# Required connection configs for Confluent Cloud Schema Registry
schema.registry.url={{ SR_URL }}
basic.auth.credentials.source=USER_INFO
basic.auth.user.info={{ SR_API_KEY }}:{{ SR_API_SECRET }}

```
> ⚠ **Warning:** Don't directly copy-paste the above configuration. Copy it from the Confluent Cloud Console so that it includes your credentials.
>
> Keep one blank line before and after the block of text above, so that we can concatenate it to other text if we want (as we will be doing later).

## Add application & producer properties
❼ `configuration/dev.properties :`

```
key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer
acks=all

#Properties below this line are specific to code in this application
input.topic.name=input-topic
output.topic.name=output-topic

```

> ℹ Meaning of the dev.properties file:
> - `key.serializer` and `value.serializer` . Tells the producer how to treat the key and the value when serializing. i.e. serialize them as strings.
> - `acks`. Tells the producer to block on the send() method call, until the message is completely committed to the broker. i.e. min in-sync replicas have acknowledged the message.
> - `output.topic.name`. The topic that we will be producing messages to.

❽ Combine the above two configs:
```
cat configuration/ccloud.properties >> configuration/dev.properties
```

❿ Create data to produce to Kafka in input.txt

```
1-value
2-words
3-All Streams
4-Lead to
5-Kafka
6-Go to
7-Kafka Summit
8-How can
9-a 10 ounce
10-bird carry a
11-5lb coconut
```

## Create the Project
❻ Use the Gradle build file, named `build.gradle` :

Obtain the gradle wrapper:
```
gradle wrapper
```
❾ Use the `KafkaProducerApplication.java` file which has the `KafkaProducerApplication` class.

## Compile and Run
⓫
Compile:

```
./gradlew shadowJar
```

Run:

```
java -jar build/libs/kafka-producer-application-standalone-0.0.1.jar configuration/dev.properties input.txt
```

Output should look like:

```
Offsets and timestamps committed in batch from input.txt
Record written to offset 0 timestamp 1597352120029
Record written to offset 1 timestamp 1597352120037
Record written to offset 2 timestamp 1597352120037
Record written to offset 3 timestamp 1597352120037
Record written to offset 4 timestamp 1597352120037
Record written to offset 5 timestamp 1597352120037
Record written to offset 6 timestamp 1597352120037
Record written to offset 7 timestamp 1597352120037
Record written to offset 8 timestamp 1597352120037
Record written to offset 9 timestamp 1597352120037
Record written to offset 10 timestamp 1597352120038
```