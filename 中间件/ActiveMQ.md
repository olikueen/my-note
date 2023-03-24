[toc]

```shell
# activemq 三个节点
192.168.101.141 k8s-master01
192.168.101.142 k8s-master02
192.168.101.143 k8s-master03
```

# Zookeeper

## 部署

```shell
# 安装jdk环境
yum install -y java-1.8.0-openjdk

tar -xf apache-zookeeper-3.7.0-bin.tar.gz -C /usr/local/
cd /usr/local/apache-zookeeper-3.7.0-bin
```

**修改配置文件**

```shell
cd conf
cp zoo_sample.cfg zoo.cfg

vim zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper
clientPort=2181
# 集群全部节点的IP
server.0=192.168.101.141:2888:3888
server.1=192.168.101.142:2888:3888
server.2=192.168.101.143:2888:3888
# tickTime：基本事件单元，这个时间是作为Zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔，每隔tickTime时间就会发送一个心跳；最小 的session过期时间为2倍tickTime
# dataDir：存储内存中数据库快照的位置，除非另有说明，否则指向数据库更新的事务日志。注意：应该谨慎的选择日志存放的位置，使用专用的日志存储设备能够大大提高系统的性能，如果将日志存储在比较繁忙的存储设备上，那么将会很大程度上影像系统性能
# client：监听客户端连接的端口
# initLimit：允许follower连接并同步到Leader的初始化连接时间，以tickTime为单位。当初始化连接时间超过该值，则表示连接失败
# syncLimit：表示Leader与Follower之间发送消息时，请求和应答时间长度。如果follower在设置时间内不能与leader通信，那么此follower将会被丢弃
# server.A=B:C:D	A：其中 A 是一个数字，表示这个是服务器的编号；	B：是这个服务器的 ip 地址；	C：Zookeeper服务器之间的通信端口；	D：Leader选举的端口。
```

**创建myid**

在配置文件的 dataDir 指定的位置下 创建myid文件, 内容是 server. 到 = 中间的数字

```shell
mkdir -pv /data/zookeeper
cd /data/zookeeper
echo 0 > myid
```

**把配置同步到 zookeeper 的三个节点**

## 运行

**启动三个节点zookeeper**

```shell
cd /usr/local/apache-zookeeper-3.7.0-bin/bin
./zkServer.sh start
```

**查看节点状态, 三个节点会有不同的状态 Mode**

```shell
[root@k8s-master01 bin]# ./zkServer.sh status
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /usr/local/apache-zookeeper-3.7.0-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower

[root@k8s-master02 bin]# ./zkServer.sh status
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /usr/local/apache-zookeeper-3.7.0-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower

[root@k8s-master03 bin]# ./zkServer.sh status
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /usr/local/apache-zookeeper-3.7.0-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader
```

**关闭**

```shell
./zkServer.sh stop
```

## 使用

**zkCli.sh 连接zookeeper集群**

```shell
```

# Activemq

## 部署

```shell
tar -xf apache-activemq-5.15.15-bin.tar.gz -C /usr/local/
cd /usr/local/apache-activemq-5.15.15/
```

## 配置

```shell
vim conf/activemq.xml
# broker标签: 修改 bokerName 为 active-cluster
 40     <broker xmlns="http://activemq.apache.org/schema/core" brokerName="active-cluster" dataDirectory="${activemq.data}">
# persistenceAdapter标签: 配置zookeeper; 其余两个节点, 需要调整 bind端口 和 hostname
 81         <persistenceAdapter>
 82             <!--<kahaDB directory="${activemq.data}/kahadb"/>-->
 83             <replicatedLevelDB
 84                 directory="${activemq.data}/leveldb"
 85                 replicas="3"
 86                 bind="tcp://0.0.0.0:62621"
 87                 zkAddress="192.168.101.141:2181,192.168.101.142:2181,192.168.101.143:2181"
 88                 hostname="192.168.101.141"
 89                 zkPath="/activemq/leveldb-stores"/>
 90         </persistenceAdapter>
# transportConnectors 第一个节点不用修改; 其余两个节点, 修改 name="openwire" 的端口 +1
118         <transportConnectors>
119             <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
120             <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
121             <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
122             <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
123             <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
124             <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
125         </transportConnectors>
```

## 运行

**启动三个节点activemq**

```shell
./bin/activemq start
# 查看运行日志
tailf data/activemq.log
```





















