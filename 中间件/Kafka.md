官网

http://kafka.apache.org/

下载安装包

https://mirrors.bfsu.edu.cn/apache/kafka/2.6.0/kafka_2.13-2.6.0.tgz

### 安装Kafka

**解压缩就行**

```
tar -xvf  kafka_2.13-2.6.0.tgz -C /usr/local/
ln -sv  /usr/local/kafka_2.13-2.6.0 /usr/local/kafka  
```

**安装Java8以上版本jdk**

 ```
 dnf install -y  java-11-openjdk
 ```

### 启动Kafka

 **启动zookeeper** 

```
cd /usr/local/kafka
./bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

**启动kafka**

```
./bin/kafka-server-start.sh  -daemon config/server.properties
```

#### 使用console生产者客户端

```
./bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092               
```

**发送消息**

```
 >test message01
 >test message02
```

#### 使用console消费者客户端

 ```
 ./bin/kafka-console-consumer.sh --topic test --from-beginning  --bootstrap-server localhost:9092   test message01
 test message02     
 ```

### 关闭kafka

**关闭kafka**

```
./bin/kafka-server-stop.sh
```

**关闭zookeeper**

```
./bin/zookeeper-server-stop.sh
```

