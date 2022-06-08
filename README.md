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

## Initialise the project
❷

Make the local directory for the project
```
mkdir creating-first-apache-kafka-streams-application && cd creating-first-apache-kafka-streams-application
```
Make a directory for configuration data:
```
mkdir configuration
```
❸ Write config info in files
`configuration/ccloud.properties`

It looks like this:
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

> Keep one blank line before and after the block of text above, so that we can concatenate it to other text if we want (as we will be doing later).



## Add application & cloud connection properties
❺ `configuration/dev.properties :`

```
application.id=kafka-streams-101

input.topic.name=random-strings
output.topic.name=tall-random-strings

key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer
```

> ℹ Meaning of the dev.properties file:
> - `application.id`: 
> - `input.topic.name` and `output.topic.name`. The topic that we will be consuming from and producing output to.
> - `key.serializer` and `value.serializer` . Tells the producer how to treat the key and the value when serializing. i.e. serialize them as strings.

❻ Combine the above two configs:
```
cat configuration/ccloud.properties >> configuration/dev.properties
```

## Create the Project
❺ Use the Gradle build file, named `build.gradle`
>  ⚠ **Warning:** Make sure to change line 22 from
>
> `mainClass.set("io.confluent.developer.KafkaStreamsApplication")`
>
> to
>
> `mainClassName = "io.confluent.developer.KafkaStreamsApplication"`

Obtain the gradle wrapper:
```
gradle wrapper --gradle-version 7.4.2
```
❼ Use the `Util.java` file which has the `Util` class containing the `Randomizer` class, and the `KafkaStreamsApplication.java` file with the `KafkaStreamsApplication` class.


## Compile and Run
❾
Compile:

```
./gradlew shadowJar
```

Run:

```
java -jar build/libs/creating-first-apache-kafka-streams-application-*.jar configuration/dev.properties
```

Output should look like:
```
[faker-StreamThread-1] INFO io.confluent.developer.KafkaStreamsApplication - Observed event: Chuck Norris can recite π. Backwards.
[faker-StreamThread-1] INFO io.confluent.developer.KafkaStreamsApplication - Transformed event: CHUCK NORRIS CAN RECITE Π. BACKWARDS.

[faker-StreamThread-1] INFO io.confluent.developer.KafkaStreamsApplication - Observed event: The class object inherits from Chuck Norris.
[faker-StreamThread-1] INFO io.confluent.developer.KafkaStreamsApplication - Transformed event: THE CLASS OBJECT INHERITS FROM CHUCK NORRIS.

[faker-StreamThread-1] INFO io.confluent.developer.KafkaStreamsApplication - Observed event: Chuck Norris' preferred IDE is hexedit.
[faker-StreamThread-1] INFO io.confluent.developer.KafkaStreamsApplication - Transformed event: CHUCK NORRIS' PREFERRED IDE IS HEXEDIT.

```

