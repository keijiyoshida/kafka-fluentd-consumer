# Kafka Consumer for Fluentd

This is a simple Java application to consume the data from Kafka, and forward to Fluentd.

## Build

Use gradle.

    $ gradle jarInAllDependencies

## Run

### Run Kafka

If you haven't run Kafka, please start Kafka. If you want to test Kafka locally, please follow the step described at [Quickstart](http://kafka.apache.org/documentation.html#quickstart).

    # start zookeeper
    $ bin/zookeeper-server-start.sh config/zookeeper.properties
    
    # start kafka
    $ bin/kafka-server-start.sh config/server.properties

Then, please create the topic named `test`

    # create test topic
    $ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

After that, please add a couple of messages into `test` topic. Please make sure message is formatted as JSON.

    # send multiple messages
    $ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test 
    {"a": 1}
    {"a": 1, "b": 2}

You can confirm messages were submitted correctly with this command.

    $ bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
    {"a": 1}
    {"a": 1, "b": 2}

### Run Kafka Consumer for Fluentd

Please modify `config/fluentd-consumer.properties` with an appropriate configuration. Especially don't forget to change to `topic=test`. Finally please launch the process like this. 2nd argument specifies the number of threads.

    $ java -jar build/libs/kafka-fluentd-consumer-all-0.0.1.jar config/fluentd-consumer.properties 1

This will forward logs to Fluentd (localhost:24224).

## Run Kafka Consumer for Fluentd via in_exec

A couple of users has been asking to host consumer as a child process of Fluentd. In that case, 

    <source>
      type forward
    </source>
    
    <source>
      type exec
      command java -jar /path/to/kafka-fluentd-consumer-all-0.0.1.jar /path/to/config/fluentd-consumer.properties 1
      tag dummy
      format json
    </source>