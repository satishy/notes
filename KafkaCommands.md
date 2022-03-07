



## Kafka frequent commands SET 1

Assuming that the following environment variables are set:
- `KAFKA_HOME` where Kafka is installed on local machine (e.g. `/opt/kafka`)
- `ZK_HOSTS` identifies running zookeeper ensemble, e.g. `ZK_HOSTS=192.168.0.99:2181`
- `KAFKA_BROKERS` identifies running Kafka brokers, e.g. `KAFKA_BROKERS=192.168.0.99:9092`

## Server

Start Zookepper and Kafka servers

    $KAFKA_HOME/bin/zookeeper-server-start.sh -daemon
    $KAFKA_HOME/bin/kafka-server-start.sh -daemon config/server.properties

Stop Kafka and Zookeeper servers

    $KAFKA_HOME/bin/kafka-server-stop.sh
    $KAFKA_HOME/bin/zookeeper-server-stop.sh

## Topics 

List topics

    $KAFKA_HOME/bin/kafka-topics.sh --zookeeper $ZK_HOSTS --list

Create topic

    $KAFKA_HOME/bin/kafka-topics.sh --zookeeper $ZK_HOSTS --create --topic $TOPIC_NAME --replication-factor 3 --partitions 3 

Topic-level configuration 

    kafka-configs.sh --zookeeper localhost:2181 --entity-type topics  --describe
    kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name $TOPIC_NAME --describe
    # Activate log compaction for the topic
    kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name $TOPIC_NAME --alter --add-config cleanup.policy=compact,delete.retention.ms=604800000

## Producer / Consumer

**Send console input to topic**

    $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list $KAFKA_BROKERS --topic hubble_stream

**Send data from file to topic**

    cat hubble_stream.dump.txt | $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list $KAFKA_BROKERS --topic hubble_stream

**Read data from topic to console**

    # ----- new way (kafka 0.10) ------
    $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server $KAFKA_BROKERS --topic mytopic   
    $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server $KAFKA_BROKERS --topic mytopic --from-beginning --max-messages 100


**Consume/Produce between 2 Kafka clusters**
    
    ./kafka-console-consumer.sh --bootstrap-server $KAFKA_BROKERS_1 --topic mytopic | $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list $KAFKA_BROKERS_2 --topic mytopic


**To watch consumers per topic/partition**

    CONSUMER_ID=$(echo "dump"|nc localhost 2181|grep "\/consumers\/"|head -n 1|awk -F'/' '{print $3}')
    $KAFKA_HOME/bin/kafka-consumer-offset-checker.sh --zookeeper $ZK_HOSTS --topic hubble_stream --group $CONSUMER_ID
    
    # Watch continuosly
    watch -n 3 $KAFKA_HOME/bin/kafka-consumer-offset-checker.sh --zookeeper $ZK_HOSTS --topic hubble_stream --group $CONSUMER_ID   
    
The expected output will be something like this

    Group           Topic                          Pid Offset          logSize         Lag             Owner
    console-consumer-27925 hubble_stream                  0   100000          100000          0               console-consumer-27925_m-C02RJ1CAG8WM.local-1481151487426-f16bdb41-0
    console-consumer-27925 hubble_stream                  1   200000          200000          0               console-consumer-27925_m-C02RJ1CAG8WM.local-1481151487426-f16bdb41-0
    console-consumer-27925 hubble_stream                  2   200000          200000          0               console-consumer-27925_m-C02RJ1CAG8WM.local-1481151487426-f16bdb41-0
    console-consumer-27925 hubble_stream                  3   100000          100000          0               console-consumer-27925_m-C02RJ1CAG8WM.local-1481151487426-f16bdb41-0
    console-consumer-27925 hubble_stream                  4   0               0               0               console-consumer-27925_m-C02RJ1CAG8WM.local-1481151487426-f16bdb41-0
    console-consumer-27925 hubble_stream                  5   0               0               0               console-consumer-27925_m-C02RJ1CAG8WM.local-1481151487426-f16bdb41-0
    
    
