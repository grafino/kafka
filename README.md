#### Kafka Setup


#### Simulated production deployment

- 3 node cluster in distributed mode
- Zookeeper install in ensemble mode
- Kafka install in distributed mode

#### Setup OS and Java

1. ###### Update OS

```
$ yum update -y
```

2. ###### Download Oracle Java

```
$ wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" \
"http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz"
```

3. ###### Untar and move to /opt/java

```
$ tar xvfz jdk-8u131-linux-x64.tar.gz
$ mv jdk1.8.0_131/ /opt/java
```


4. ###### Set java in environment

```
$ vi /etc/environment

export JAVA_HOME=/opt/java
export PATH=$PATH:$JAVA_HOME/bin
```

5. ###### Set hosts file

```
#KAFKA AND ZOOKEEPER HOSTS
192.168.50.21 broker01.kafka.local broker01 kafka1 znode1
192.168.50.22 broker02.kafka.local broker02 kafka2 znode2
192.168.50.23 broker03.kafka.local broker03 kafka3 znode3
```

#### Install ZooKeeper

http://zookeeper.apache.org/

1. ###### Download Zookeeper

```
$ wget http://www-us.apache.org/dist/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
```

2. ###### untar and move to /opt/zookeeper

```
$ tar xvfz zookeeper-3.4.10.tar.gz
$ mv zookeeper-3.4.10 /opt/zookeeper
```

3. ###### adduser zookeeper

```
$ adduser zookeeper
```

4. ###### create dirs and add permissions

```
$ mkdir /zookeeper/snap
$ mkdir /zookeeper/logs

$ chown -R zookeeper:zookeeper /opt/zookeeper
$ chown -R zookeeper:zookeeper /zookeeper
```

5. ###### Configure Zookeeper

```
vi /opt/zookeeper/conf/zoo.cfg
```

```
dataDir=/zookeeper/snap
dataLogDir=/zookeeper/logs
clientPort=2181
maxClientCnxns=20
initLimit=5
syncLimit=2
server.1=znode1:2888:3888
server.2=znode2:2888:3888
server.3=znode3:2888:3888
```

7. ###### Set the id files in all servers

echo (Server Number) > /zookeeper/snap/myid


8. ###### Start Zookeeper

```
zkServer.sh start
```

OR

```
/opt/kafka/bin/zookeeper-server-start -daemon /opt/kafka/etc/kafka/zookeeper.properties
```

9. ###### Test zookeeper

```
[kafka@broker01 ~]$ jps
4297 Jps
4266 QuorumPeerMain
```

AND / OR

```
/opt/kafka/bin/zookeeper-shell znode1:2181
```

#### Install kafka

* https://kafka.apache.org/
* https://www.confluent.io

1. ###### Add user kafka for running

```
$ adduser kafka
```

2. ###### Download kafka

```
$ wget http://packages.confluent.io/archive/3.2/confluent-oss-3.2.2-2.11.tar.gz
```

3. ###### Untar and move to /opt/kafka
```
$ tar xvfz confluent-oss-3.2.2-2.11.tar.gz
$ mv confluent-3.2.2/ /opt/kafka
```

4. ###### Change permissions

```
$ chown -R kafka:kafka /opt/kafka/
```


5. ###### Create log dirs and set permissions

```
$ mkdir log.dirs=/kafka/
$ chown -R kafka:kafka /kafka/
```

6. ###### Kafka configurations

```
$ vi /opt/kafka/etc/kafka/server.properties
```

```
broker.id=1
delete.topic.enable=true
default.replication.factor=3
listeners=PLAINTEXT://kafka1:9092
advertised.listeners=PLAINTEXT://kafka1:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/kafka/
num.partitions=3
num.recovery.threads.per.data.dir=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=znode1:2181,znode2:2181,znode3:2181
zookeeper.connection.timeout.ms=5000
confluent.support.metrics.enable=true
confluent.support.customer.id=anonymous
```



5. #### START kafka

```
$/opt/kafka/bin/kafka-server-start -daemon /opt/kafka/etc/kafka/server.properties
```


### Kafka tools

1. ###### Create Topic

```
kafka-topics --zookeeper 192.168.50.21:2181 --create --replication-factor 3 --partitions 5 --topic mytopic
```

2. ###### List Topics

```
kafka-topics --zookeeper 192.168.50.21:2181 --list
```

3. ###### Delete Topic

```
kafka-topics --zookeeper 192.168.50.21:2181 --delete --topic mytopic
```

4. ###### Describe Topic

```
kafka-topics --zookeeper 192.168.50.21:2181 --describe --topic mytopic
```

* "leader" is the node responsible for all reads and writes for the given partition. Each node will be the leader for a randomly selected portion of the partitions.

* "replicas" is the list of nodes that replicate the log for this partition regardless of whether they are the leader or even if they are currently alive.

* "isr" is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive and caught-up to the leader.


5. ###### Kafka Console Producer
```
kafka-console-producer --broker-list kafka1:9092,kafka2:9092,kafka3:9092 --topic mytopic
```

6. ###### Kafka Console Consumer

```
kafka-console-consumer --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092  --topic mytopic2 --from-beginning --consumer-property group.id=consumer-group
```


7. ###### View Offsets

- -time -1 = lasgest Offset
- -time -2 = smallest Offset
- topic:partitionId:offset

```
kafka-run-class kafka.tools.GetOffsetShell --broker-list kafka1:9092,kafka2:9092,kafka3:9092 -topic mytopic  --time -1

kafka-run-class kafka.tools.GetOffsetShell --broker-list kafka1:9092,kafka2:9092,kafka3:9092 -topic mytopic  --time -2
```

8. ###### View Consumer per group
```
kafka-consumer-groups --bootstrap-server kafka1:9092 --describe --group consumer-group
```

#####

```
$ vi /etc/environment

export JAVA_HOME=/opt/java
export ZOOKEEPER_HOME=/opt/zookeeper
export KAFKA_HOME=/opt/kafka
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin:$KAFKA_HOME/bin
```


#### Zookeeper connect to client

```
$ zkCli.sh -server znode1:2181

> ls /
````




#### Schema Registry - For later

http://docs.confluent.io/current/schema-registry/docs/intro.html#installation


Schema Registry provides a serving layer for your metadata. It provides a RESTful interface for storing and retrieving Avro schemas. It stores a versioned history of all schemas, provides multiple compatibility settings and allows evolution of schemas according to the configured compatibility setting. It provides serializers that plug into Kafka clients that handle schema storage and retrieval for Kafka messages that are sent in the Avro format.



#### Kafka Security - For later

7. Open Ports


Component	Default Port
Zookeeper	2181
Apache Kafka brokers 9092
Schema Registry REST API 8081
REST Proxy	8082
Kafka Connect REST API	8083
Confluent Control Center	9021

8. You should increase your file descriptor
