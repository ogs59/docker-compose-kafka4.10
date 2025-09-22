## docker-compose-kafka4.10

前言
Docker-compose快速部署kafka集群，镜像版本7.7.1对镜kafka4.1，本次部署基于kraft

## 一.


## 二. 部署须知

项目文件夹权限问题，第一次docker-compose up -d 必定失败，给目录赋权。

```
chown -R 1000.1000 data config
chmod 755 data

```

## 三. 预处理

kraft版启动时必须拿到 KAFKA_CLUSTER_ID 环境变量，或者已经在数据目录中发现过 meta.properties 文件中的 cluster.id

通过一下命令为broker创建meta.properties
```
先生成uuid

[root@anolios-01 kafka4.1]# docker run --rm   -v $PWD/config/broker-1.properties:/etc/kafka/server.properties   confluentinc/cp-kafka:7.7.1   kafka-storage random-uuid
ohEfzflmT6-TloTKzfp5YA
[root@anolios-01 kafka4.1]# 
[root@anolios-01 kafka4.1]# 


broker1:
docker run --rm -v $PWD/data/broker-1:/var/lib/kafka/data  -v $PWD/config/broker-1.properties:/etc/kafka/server.properties  confluentinc/cp-kafka:7.6.0   kafka-storage format --cluster-id ohEfzflmT6-TloTKzfp5YA --config /etc/kafka/server.properties

broker2:
docker run --rm -v $PWD/data/broker-1:/var/lib/kafka/data  -v $PWD/config/broker-1.properties:/etc/kafka/server.properties  confluentinc/cp-kafka:7.6.0   kafka-storage format --cluster-id ohEfzflmT6-TloTKzfp5YA --config /etc/kafka/server.properties

broker3:
docker run --rm -v $PWD/data/broker-1:/var/lib/kafka/data  -v $PWD/config/broker-1.properties:/etc/kafka/server.properties  confluentinc/cp-kafka:7.6.0   kafka-storage format --cluster-id ohEfzflmT6-TloTKzfp5YA --config /etc/kafka/server.properties


[root@anolios-01 kafka4.1]# ls data/broker-{1..3}/
data/broker-1/:
bootstrap.checkpoint  meta.properties

data/broker-2/:
bootstrap.checkpoint  meta.properties

data/broker-3/:
bootstrap.checkpoint  meta.properties

```

新生成的文件记得chown -R 1000.1000 data/broker-{1..3}，否则是启动失败。

## 环境变量配置

1. uuid这个坑比较大，启动了数十次秒失败，查看日志uuid获取不到，弄的焦头烂额，跟ai无数次拉扯，最后才招了，通过CLUSTER_ID 获取的，跟KAFKA_CLUSTER_ID没半毛钱关系

2. 容器的端口映射，记得把 ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.125:9092映射出来，因为kafka-ui和kafka-exporter都需要通过这个端口获取数据。

   项目涉及的环境变量如下，部署之前还是仔细看一下

```
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_NODE_ID: 1
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-1:19092,2@kafka-2:19093,3@kafka-3:19094
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:19092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.125:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_LOG_DIRS: /var/lib/kafka/data
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_CLUSTER_ID: "ohEfzflmT6-TloTKzfp5YA"
      CLUSTER_ID: "ohEfzflmT6-TloTKzfp5YA"
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
```


## 启动

```
[root@anolios-01 docker-compose-kafka4.10]# 
[root@anolios-01 docker-compose-kafka4.10]# docker compose up -d 
[+] Running 5/5
 ✔ Network docker-compose-kafka410_kafka-net  Created                                                                                                                                                 0.1s 
 ✔ Container kafka-2                          Started                                                                                                                                                 0.3s 
 ✔ Container kafka-3                          Started                                                                                                                                                 0.2s 
 ✔ Container kafka-ui                         Started                                                                                                                                                 0.2s 
 ✔ Container kafka-1                          Started                                                                                                                                                 0.3s 
[root@anolios-01 docker-compose-kafka4.10]# 
[root@anolios-01 docker-compose-kafka4.10]# 
[root@anolios-01 docker-compose-kafka4.10]# docker ps 
CONTAINER ID   IMAGE                           COMMAND                   CREATED         STATUS         PORTS                                              NAMES
33ff1c9a46e4   confluentinc/cp-kafka:7.7.1     "/etc/confluent/dock…"   3 seconds ago   Up 2 seconds   0.0.0.0:9092->9092/tcp, 0.0.0.0:19092->19092/tcp   kafka-1
fe6ff668c345   confluentinc/cp-kafka:7.7.1     "/etc/confluent/dock…"   3 seconds ago   Up 2 seconds   0.0.0.0:9094->9092/tcp, 0.0.0.0:19094->19092/tcp   kafka-3
4dbdcf4d6e14   confluentinc/cp-kafka:7.7.1     "/etc/confluent/dock…"   3 seconds ago   Up 2 seconds   0.0.0.0:9093->9092/tcp, 0.0.0.0:19093->19092/tcp   kafka-2
037760129f17   provectuslabs/kafka-ui:latest   "/bin/sh -c 'java --…"   3 seconds ago   Up 3 seconds   0.0.0.0:8080->8080/tcp                             kafka-ui
[root@anolios-01 docker-compose-kafka4.10]# 
```


## 简单测试




