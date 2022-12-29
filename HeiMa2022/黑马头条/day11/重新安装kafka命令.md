关闭原来的zookeeper和kafka

```shell
docker stop zookeeper
docker stop kafka
```

重启启动zookeeper

```shell
docker run -d \
--restart=always \
--log-driver json-file \
--log-opt max-size=100m \
--log-opt max-file=2  \
--name zookeeper2 \
-p 2181:2181 \
-v /etc/localtime:/etc/localtime \
wurstmeister/zookeeper
```

重启启动kafka，其中

```shell
docker run -d \
--restart=always \
--log-driver json-file \
--log-opt max-size=100m \
--log-opt max-file=2 \
--name kafka2 \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=192.168.200.130/kafka \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.200.130:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
-v /etc/localtime:/etc/localtime \
wurstmeister/kafka
```








