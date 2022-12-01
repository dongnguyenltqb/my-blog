---
title: Kafka with docker compose
date: 2022/10/12
description: This article assume that you have basic knowledge about Kafka, and Docker Compose, for development environment, there is no strict requirement about high availability and durability, we will try to make our cluster same as production, staging environment as much as possible.
tag: kafka,docker-compose
author: You
---

This article assume that you have basic knowledge about Kafka, and Docker Compose, for development environment, there is no strict requirement about high availability and durability, we will try to make our cluster same as production, staging environment as much as possible.

Our infrastructure architecture is very simple, there is VM that run the application and the other VM that running the database, caching engine, search engine, etc…. The application will connect to Kafka using private ip address.

First create `kafka-cluster` directory where we store needed assets to run docker container.

```
[root@localhost ~]# mkdir kafka-cluster
[root@localhost ~]# cd kafka-cluster
```

Next, download and extract kafka to the current directory.

```
[root@localhost kafka-cluster]# wget https://downloads.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz

kafka_2.13-3.3.1.tgz    100%[============================>] 100.19M  2.72MB/s    in 45s

2022-11-15 10:22:59 (2.24 MB/s) - ‘kafka_2.13-3.3.1.tgz’ saved [105053134/105053134]

[root@localhost kafka-cluster]# tar -xf kafka_2.13-3.3.1.tgz
[root@localhost kafka-cluster]# ll
total 102592
drwxr-xr-x. 7 root root       105 Sep 30 02:06 kafka_2.13-3.3.1
-rw-r--r--. 1 root root 105053134 Oct  3 06:05 kafka_2.13-3.3.1.tgz
[root@localhost kafka-cluster]#
```

We will create `configfiles` and `dockerfiles` to store the `properties` file for `zookeeper` `kafka` and `Dockerfile` for them.

```
[root@localhost kafka-cluster]# mkdir configfiles dockerfiles
```

Let prepare config file for `zookeeper` , our zookeeper is private by default and do not expose any port to the host. The file is located at `configfiles/zookeeper.properties` . We will mount this file to container when it’s running.

```
# the directory where the snapshot is stored.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=14000
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080
```

Next step, we will create `Dockerfile` for zookeeper to build `zookeeper` image. The file will be located at `dockerfiles/ZookeeperDockerfile`

```
FROM openjdk:20-slim-buster
WORKDIR /app
COPY . .
CMD ./bin/zookeeper-server-start.sh config/zookeeper.properties
```

Next we will create `docker-compose.yml` to setup `zookeeper` service, volume and mount configuration file to container. This file will be located at current directory.

```
version: "3.9"

networks:
  default:
    name: kafka
    driver: bridge

volumes:
  zookeeperdata:

services:
  zookeeper:
    build:
      context: kafka_2.13-3.3.1
      dockerfile: ../dockerfiles/ZookeeperDockerfile
    volumes:
      - zookeeperdata:/tmp/zookeeper
      - ./configfiles/zookeeper.properties:/app/config/zookeeper.properties
```

Now the current directory tree look like.

```
[root@localhost kafka-cluster]# tree .
.
├── configfiles
│   └── zookeeper.properties
├── docker-compose.yml
├── dockerfiles
│   └── ZookeeperDockerfile
├── kafka_2.13-3.3.1
│   ├── bin
│   │   ├── connect-distributed.sh
│   │   ├── connect-mirror-maker.sh
│   │   ├── connect-standalone.sh
│   │   ├── kafka-acls.sh
│   │   ├── kafka-broker-api-versions.sh
│   │   ├── kafka-cluster.sh
│   │   ├── kafka-configs.sh
│   │   ├── kafka-console-consumer.sh
│   │   ├── kafka-console-producer.sh
│   │   ├── kafka-consumer-groups.sh
│   │   ├── kafka-consumer-perf-test.sh
│   │   ├── kafka-delegation-tokens.sh
│   │   ├── kafka-delete-records.sh
│   │   ├── kafka-dump-log.sh
│   │   ├── kafka-features.sh
│   │   ├── kafka-get-offsets.sh
```

Now we can able to build and run `zookeeper` container.

```
[root@localhost kafka-cluster]# docker compose up -d --build zookeeper
[+] Building 3.0s (8/8) FINISHED
 => [internal] load build definition from ZookeeperDockerfile                                                                                                    0.1s
 => => transferring dockerfile: 100B                                                                                                                             0.0s
 => [internal] load .dockerignore                                                                                                                                0.1s
 => => transferring context: 2B                                                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/openjdk:20-slim-buster                                                                                        2.7s
 => [internal] load build context                                                                                                                                0.1s
 => => transferring context: 23.22kB                                                                                                                             0.0s
 => [1/3] FROM docker.io/library/openjdk:20-slim-buster@sha256:07c677b7e30d0e09a97fb6c882a0ce23cdc2c3ae12d2cee6d836a29122e4e903                                  0.0s
 => CACHED [2/3] WORKDIR /app                                                                                                                                    0.0s
 => CACHED [3/3] COPY . .                                                                                                                                        0.0s
 => exporting to image                                                                                                                                           0.1s
 => => exporting layers                                                                                                                                          0.0s
 => => writing image sha256:ef70a2dd46b916a12f54004e8e29439c86b539f41440affee757653d2eb80404                                                                     0.0s
 => => naming to docker.io/library/kafka-cluster-zookeeper                                                                                                       0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
[+] Running 1/1
 ⠿ Container kafka-cluster-zookeeper-1  Started                                                                                                                  0.4s
[root@localhost kafka-cluster]#
```

You can see the log to make sure it’s running.

```
[root@localhost kafka-cluster]# docker compose ps
NAME                        COMMAND                  SERVICE             STATUS              PORTS
kafka-cluster-zookeeper-1   "/bin/sh -c './bin/z…"   zookeeper           running
[root@localhost kafka-cluster]# docker compose logs -f zookeeper
```

Next we will setup three kafka brokers, let assume let the private ip address of our virtual machine is `10.0.2.15` . We will have 3 container that has name `leesin` `garen` and `temo` , each one will expose different port on host machine `4000` `5000` and `6000` .

The `dockerfile` for each broker will look like this, the file is located at `dockerfiles/KafkaDockerfile`

```
FROM openjdk:20-slim-buster
WORKDIR /app
COPY . .
CMD ./bin/kafka-server-start.sh config/server.properties
```

For each broker, we will prepare their `server.properties` file, the port that they will listen for inter-broker communicate and the port for receiving connection from external client.

For `leesin` broker, we put the `server.properties` into `configfiles/leesin/server.properties` and mount the configuration file to the container when it start.

The content is look like this.

```
broker.id=1
zookeeper.connect=zookeeper:14000
zookeeper.connection.timeout.ms=18000
sasl.enabled.mechanisms=PLAIN
sasl.mechanism.inter.broker.protocol=PLAIN
listener.name.internal.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
 username="admin" \
 password="admin-secret" \
 user_admin="admin-secret";

listener.name.external.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
     username="admin" \
     password="admin-secret" \
 user_admin="admin-secret";

listeners=INTERNAL://:9092,EXTERNAL://:4000
advertised.listeners=INTERNAL://leesin:9092,EXTERNAL://10.0.2.15:4000
inter.broker.listener.name=INTERNAL
listener.security.protocol.map=INTERNAL:SASL_PLAINTEXT,EXTERNAL:SASL_PLAINTEXT
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kafka-logs
num.partitions=3
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
group.initial.rebalance.delay.ms=0
```

Two thing to keep eyes on is the `listeners` value and `advertised.listeners` .

```
listeners=INTERNAL://:9092,EXTERNAL://:4000
advertised.listeners=INTERNAL://leesin:9092,EXTERNAL://10.0.2.15:4000
```

For now, our `docker-compose.yml` is look like this.

```
version: "3.9"

networks:
  default:
    name: kafka
    driver: bridge

volumes:
  leesindata:
  zookeeperdata:

services:
  zookeeper:
    build:
      context: kafka_2.13-3.3.1
      dockerfile: ../dockerfiles/ZookeeperDockerfile
    volumes:
      - zookeeperdata:/tmp/zookeeper
      - ./configfiles/zookeeper.properties:/app/config/zookeeper.properties

  leesin:
    build:
      context: kafka_2.13-3.3.1
      dockerfile: ../dockerfiles/KafkaDockerfile
    volumes:
      - leesindata:/tmp/kafka-logs
      - ./configfiles/leesin/server.properties:/app/config/server.properties
    ports:
      - 4000:4000
```

Before doing that, if you set up a firewall, make sure you allow the incoming traffic from port `4000` `5000` and `6000` .

![img](https://miro.medium.com/max/875/1*wuT5EDuHzZhNcxgF04BeMA.png)

open incoming traffic to kafka broker

Now start the `leesin` broker.

```
[root@localhost kafka-cluster]# docker compose up -d leesin
```

You can check the log and see that `leesin` broker worked.

```
[root@localhost kafka-cluster]# docker compose ps
NAME                        COMMAND                  SERVICE             STATUS              PORTS
kafka-cluster-leesin-1      "/bin/sh -c './bin/k…"   leesin              running             0.0.0.0:4000->4000/tcp, :::4000->4000/tcp
kafka-cluster-zookeeper-1   "/bin/sh -c './bin/z…"   zookeeper           running
[root@localhost kafka-cluster]#
```

Keep create `server.properties` file for 2 left broker `garen` and `temo` .

The `garen` ‘s file look like this, the file is located at `configfiles/garen/server.properties` (the different from `leesin` configuration file is just `broker_id` , `listener` , `advertised.listeners.name` )

```
broker.id=2
zookeeper.connect=zookeeper:14000
zookeeper.connection.timeout.ms=18000
sasl.enabled.mechanisms=PLAIN
sasl.mechanism.inter.broker.protocol=PLAIN
listener.name.internal.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
 username="admin" \
 password="admin-secret" \
 user_admin="admin-secret";

listener.name.external.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
     username="admin" \
     password="admin-secret" \
 user_admin="admin-secret";

listeners=INTERNAL://:9092,EXTERNAL://:5000
advertised.listeners=INTERNAL://garen:9092,EXTERNAL://10.0.2.15:5000
inter.broker.listener.name=INTERNAL
listener.security.protocol.map=INTERNAL:SASL_PLAINTEXT,EXTERNAL:SASL_PLAINTEXT
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kafka-logs
num.partitions=3
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
group.initial.rebalance.delay.ms=0
```

The `temo` ‘s file look like this, the file is located at `configfiles/temo/server.properties`

```
broker.id=3
zookeeper.connect=zookeeper:14000
zookeeper.connection.timeout.ms=18000
sasl.enabled.mechanisms=PLAIN
sasl.mechanism.inter.broker.protocol=PLAIN
listener.name.internal.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
 username="admin" \
 password="admin-secret" \
 user_admin="admin-secret";

listener.name.external.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
     username="admin" \
     password="admin-secret" \
 user_admin="admin-secret";

listeners=INTERNAL://:9092,EXTERNAL://:6000
advertised.listeners=INTERNAL://temo:9092,EXTERNAL://10.0.2.15:6000
inter.broker.listener.name=INTERNAL
listener.security.protocol.map=INTERNAL:SASL_PLAINTEXT,EXTERNAL:SASL_PLAINTEXT
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/tmp/kafka-logs
num.partitions=3
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
group.initial.rebalance.delay.ms=0
```

For now the directory tree is look like.

```
[root@localhost kafka-cluster]# tree
.
├── configfiles
│   ├── garen
│   │   └── server.properties
│   ├── leesin
│   │   └── server.properties
│   ├── temo
│   │   └── server.properties
│   └── zookeeper.properties
├── docker-compose.yml
├── dockerfiles
│   ├── KafkaDockerfile
│   └── ZookeeperDockerfile
├── kafka_2.13-3.3.1
│   ├── bin
│   │   ├── connect-distributed.sh
│   │   ├── connect-mirror-maker.sh

│   │   ├── kafka-server-start.sh
│   │   ├── zookeeper-security-migration.sh
│   │   ├── zookeeper-server-start.sh
│   │   ├── zookeeper-server-stop.sh
│   │   └── zookeeper-shell.sh
│   ├── config
│   │   ├── connect-console-sink.properties
│   │   ├── connect-console-source.properties
│   │   ├── connect-distributed.properties
│   │   ├── connect-file-sink.properties
```

Update `docker-compose.yml` and add `garen` `temo` service.

The file’s content is look like this.

```
version: "3.9"

networks:
  default:
    name: kafka
    driver: bridge

volumes:
  leesindata:
  garendata:
  temodata:
  zookeeperdata:

services:
  zookeeper:
    build:
      context: kafka_2.13-3.3.1
      dockerfile: ../dockerfiles/ZookeeperDockerfile
    volumes:
      - zookeeperdata:/tmp/zookeeper
      - ./configfiles/zookeeper.properties:/app/config/zookeeper.properties

  leesin:
    build:
      context: kafka_2.13-3.3.1
      dockerfile: ../dockerfiles/KafkaDockerfile
    volumes:
      - leesindata:/tmp/kafka-logs
      - ./configfiles/leesin/server.properties:/app/config/server.properties
    ports:
      - 4000:4000

  garen:
    build:
      context: kafka_2.13-3.3.1
      dockerfile: ../dockerfiles/KafkaDockerfile
    volumes:
      - garendata:/tmp/kafka-logs
      - ./configfiles/garen/server.properties:/app/config/server.properties
    ports:
      - 5000:5000

  temo:
    build:
      context: kafka_2.13-3.3.1
      dockerfile: ../dockerfiles/KafkaDockerfile
    volumes:
      - temodata:/tmp/kafka-logs
      - ./configfiles/temo/server.properties:/app/config/server.properties
    ports:
      - 6000:6000
```

Time to start them

```
[root@localhost kafka-cluster]# docker compose up -d garen temo
```

Verify these container is running.

```
[root@localhost kafka-cluster]# docker compose ps
NAME                        COMMAND                  SERVICE             STATUS              PORTS
kafka-cluster-garen-1       "/bin/sh -c './bin/k…"   garen               running             0.0.0.0:5000->5000/tcp, :::5000->5000/tcp
kafka-cluster-leesin-1      "/bin/sh -c './bin/k…"   leesin              running             0.0.0.0:4000->4000/tcp, :::4000->4000/tcp
kafka-cluster-temo-1        "/bin/sh -c './bin/k…"   temo                running             0.0.0.0:6000->6000/tcp, :::6000->6000/tcp
kafka-cluster-zookeeper-1   "/bin/sh -c './bin/z…"   zookeeper           running
[root@localhost kafka-cluster]#
```

Now, we should set up a `user interface` to check the current kafka cluster start, i will use an opensource from https://github.com/provectus/kafka-ui

In `docker-compose.yml` file, i will add this service.

```
ui:
  image: provectuslabs/kafka-ui:latest
  ports:
    - "8080:8080"
  env_file:
    - env-files/ui.env
```

The `env_files` is look like this. The file is located at `env-files/ui.env`

```
KAFKA_CLUSTERS_0_NAME=kafka-cluster
KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=10.0.2.15:4000
KAFKA_CLUSTERS_0_ZOOKEEPER=zookeeper:14000
KAFKA_CLUSTERS_0_PROPERTIES_SECURITY_PROTOCOL=SASL_PLAINTEXT
KAFKA_CLUSTERS_0_PROPERTIES_SASL_MECHANISM=PLAIN
KAFKA_CLUSTERS_0_PROPERTIES_SASL_JAAS_CONFIG=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="admin-secret";
```

Start this service.

```
[root@localhost kafka-cluster]# docker compose up ui
```

Now visit `10.0.2.15:8080` to see the cluster status.

![img](https://miro.medium.com/max/875/1*8-ErzjjW6TsuYRHBrEZ73A.png)

kafka management ui

So our cluster worked, you can download the source code here.

[dongnguyenltqb/kafka_2.13-3.3.1-docker-compose](https://github.com/dongnguyenltqb/kafka_2.13-3.3.1-docker-compose)

Thank for reading !
