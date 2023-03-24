[toc]

**Hadoop核心组件**

- Hadoop HDFS（分布式文件存储系统）：解决海量数据存储
- Hadoop YARN（集群资源管理和任务调度框架）：结局资源任务调度
- Hadoop MapReduce（分布式计算框架）：解决海量数据计算

## 1. 环境说明

```
节点: (1c 2G 100G)*3
JDK: oracle-jdk-11.0.14
Hadoop:  hadoop-3.3.2

hosts:
192.168.111.181 hadoop1
192.168.111.182 hadoop2
192.168.111.183 hadoop3
```

## 2. 安装部署

**创建用户**

```shell
useradd hadoop
echo 'qweqwe' | passwd hadoop --stdin
echo 'hadoop ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

**免密认证**

```ssh
ssh-keygen
ssh-copy-id 127.0.0.1

scp -r .ssh hadoop2:~/
scp -r .ssh hadoop3:~/
```

**安装jdk**

```shell
tar -xvf jdk-11.0.14_linux-x64_bin.tar.gz -C /usr/local/

echo 'export JAVA_HOME=/usr/local/jdk-11.0.14' >> /etc/profile
echo 'export CLASSPATH=$JAVA_HOME/lib' >> /etc/profile
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profile
. /etc/profile
```

**安装hadoop**

```shell
tar -xf hadoop-3.3.2.tar.gz -C /usr/local/

echo 'export HADOOP_HOME=/usr/local/hadoop-3.3.2' >> /etc/profile
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> /etc/profile
. /etc/profile

chown -R hadoop.hadoop /usr/local/hadoop-3.3.2
```

**时间同步(略)**

## 3. 运行模式

- **本地模式(测试)** 单机运行; 数据存储在本地磁盘
- **伪分布式(测试)** 单机运行, 但是具备hadoop集群的所有功能, 一台服务器模拟一个分布式环境; 数据存储在HDFS
- **完全分布式(生产)** 多节点运行; 数据存储在HDFS集群

### 3.1 本地模式

```shell
[hadoop@hadoop1 hadoop-3.3.2]# cd /usr/local/hadoop-3.3.2/
[hadoop@hadoop1 hadoop-3.3.2]# mkdir wcinput
[hadoop@hadoop1 hadoop-3.3.2]# cd wcinput/
[hadoop@hadoop1 wcinput]# cat <<EOF>> world.txt
> ss ss
> alec alec
> cd ls
> abc
> hahahaha
> ha
> EOF

[hadoop@hadoop1 wcinput]# cd ..
# 执行演示代码 wordcount 功能
[hadoop@hadoop1 hadoop-3.3.2]# bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount wcinput/ wcoutput
[hadoop@hadoop1 hadoop-3.3.2]# cat wcoutput/part-r-00000
```

### 3.2 完全分布式

#### 3.2.1 集群规划

|      | hadoop1                | hadoop2                          | hadoop3                         |
| ---- | ---------------------- | -------------------------------- | ------------------------------- |
| HDFS | NameNode<br />DataNode | DataNode                         | SecondaryNameNode<br />DataNode |
| YARN | NodeManager            | ResourceManager<br />NodeManager | NodeManager                     |

#### 3.2.2 配置文件

1. 核心配置文件 core-site.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <!-- 指定NameNode地址 -->
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop1:8020</value>
        </property>
        <!-- 指定Hadoop数据存储目录 -->
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/usr/local/hadoop-3.3.2/data</value>
        </property>
        <!-- 配置HDFS网页登录使用的静态用户为alec -->
        <!-- <property>
            <name>hadoop.http.staticuser.user</name>
            <value>alec</value>
        </property> -->
    </configuration>
    ```

2. HDFS配置文件 hdfs-site.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
       <!-- NameNode web端访问地址 -->
       <property>
           <name>dfs.namenode.http-address</name>
           <value>hadoop1:9870</value>
       </property>
       <!-- SecondaryNameNode web端访问地址 -->
       <property>
           <name>dfs.namenode.secondary.http-address</name>
           <value>hadoop3:9868</value>
       </property>
   </configuration>
   ```

3. YARN配置文件 yarn-site.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <!-- 指定mapreduce走shuffle -->
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <!-- 指定ResourceManager的地址 -->
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>hadoop2</value>
        </property>
        <!-- 环境变量的继承 -->
        <property>
            <name>yarn.nodemanager.env-whitelist</name>
            <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
        </property>
    </configuration>
    ```

4. MapReduce配置文件 mapred-site.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
    <configuration>
        <!-- 指定mapreduce程序运行在yarn上 -->
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    ```

5. 分发配置文件到其他服务器

   ```shell
   cd /usr/local/hadoop-3.3.2/
   scp -r hadoop/* hadoop2:/usr/local/hadoop-3.3.2/etc/hadoop/
   scp -r hadoop/* hadoop3:/usr/local/hadoop-3.3.2/etc/hadoop/
   ```

#### 3.2.3 启动集群

1. 配置workers, 同步到其他节点

   ```shell
   [hadoop@hadoop1 etc]# cat <<EOF>hadoop/workers
   > hadoop1
   > hadoop2
   > hadoop3
   > EOF
   [hadoop@hadoop1 etc]# scp hadoop/workers hadoop2:/usr/local/hadoop-3.3.2/etc/hadoop/
   [hadoop@hadoop1 etc]# scp hadoop/workers hadoop3:/usr/local/hadoop-3.3.2/etc/hadoop/
   ```

2. 启动集群

   ```
   如果是第一次启动, 需要在hadoop1节点格式化NameNode
   注意: 
   	格式化NameNode, 会产生新的集群id, 导致NameNode和DataNode集群id不一致, 集群找不到以往数据;
   	如果集群在运行过程中报错,需要重新格式化NameNode的话, 一定要先停止namenode和datanode进程, 并且要删除所有机器的data和logs目录, 然后再进行格式化;
   ```

   ```shell
   [hadoop@hadoop1 etc]# hdfs namenode -format
   ```

3. 启动HDFS

   ```shell
   [hadoop@hadoop1 hadoop-3.3.2]$ start-dfs.sh
   Starting namenodes on [hadoop1]
   Starting datanodes
   hadoop2: WARNING: /usr/local/hadoop-3.3.2/logs does not exist. Creating.
   hadoop3: WARNING: /usr/local/hadoop-3.3.2/logs does not exist. Creating.
   Starting secondary namenodes [hadoop3]
   ```


   > 报错 hadoop1: ERROR: JAVA_HOME is not set and could not be found.

   ```
   echo 'export JAVA_HOME=/usr/local/jdk-11.0.14' >> /usr/local/hadoop-3.3.2/etc/hadoop/hadoop-env.sh
   ```

   检查

   ```shell
   [hadoop@hadoop1 hadoop-3.3.2]$ jps
   11412 NameNode
   11834 Jps
   11531 DataNode
   ```

4. 在ResourceManager节点(hadoop2)启动YARN

   ```shell
   [hadoop@hadoop2 ~]$ start-yarn.sh 
   Starting resourcemanager
   Starting nodemanagers
   ```

5. 在web查看hdfs的NameNode: http://192.168.111.181:9870

6. 在web查看yarn的ResourceManager http://192.168.111.182:8088

#### 3.2.4 集群基本测试

1. 上传文件到集群

   ```shell
   [hadoop@hadoop1 ~]$ hadoop fs -mkdir /wcinput
   # http://192.168.111.181:9870/explorer.html 查看目录
   [hadoop@hadoop1 ~]$ hadoop fs -put /usr/local/hadoop-3.3.2/wcinput/world.txt /wcinput
   ```

   ![image-20220406214211825](C:/MyNote/%E5%A4%A7%E6%95%B0%E6%8D%AE/media/image-20220406214211825.png)

2. 下载文件

   ```shell
   [hadoop@hadoop1 ~]$ hadoop fs -get /wcinput/world.txt ./
   ```

3. 执行worldcount程序

   ```shell
   [hadoop@hadoop1 ~]$ hadoop jar /usr/local/hadoop-3.3.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount /wcinput /wcoutput
   ```

   ![image-20220406220113692](C:/MyNote/%E5%A4%A7%E6%95%B0%E6%8D%AE/media/image-20220406220113692.png)

#### 3.2.5 配置历史服务器

1. 配置mapred-site.xml

   ```xml
       <!-- 历史服务器server端地址 -->
       <property>
           <name>mapreduce.jobhistory.address</name>
           <value>hadoop1:10020</value>
       </property>
       <!-- 历史服务器web端地址 -->
       <property>
           <name>mapreduce.jobhistory.webapp.address</name>
           <value>hadoop1:19888</value>
       </property>
   ```

2. 分发配置

   ```shell
   scp etc/hadoop/mapred-site.xml hadoop2:/usr/local/hadoop-3.3.2/etc/hadoop/mapred-site.xml
   scp etc/hadoop/mapred-site.xml hadoop3:/usr/local/hadoop-3.3.2/etc/hadoop/mapred-site.xml
   ```

3. 启动历史服务器

   ```shell
   ssh hadoop2
   # 启动historyserver
   mapred --daemon start historyserver
   # 重启yarn
   stop-yarn.sh && start-yarn.sh
   ```

   ```shell
   [hadoop@hadoop1 hadoop-3.3.2]$ jps
   4066 NameNode
   5012 JobHistoryServer
   2120 NodeManager
   4188 DataNode
   5069 Jps
   ```

4. 查看history

   ```shell
   # 清理wcoutput
   [hadoop@hadoop1 hadoop-3.3.2]$ hadoop fs -rm -r -f /wcoutput
   
   [hadoop@hadoop1 hadoop-3.3.2]$ hadoop jar /usr/local/hadoop-3.3.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount /wcinput /wcoutput
   
   # 浏览器查看
   ```

#### 3.2.6 配置日志聚集

日志聚集: 应用运行完成以后, 将程序运行日志信息上传到HDFS系统上

注意: 开启日志聚集, 需要重启 NodeManager ResourceManager HistoryServer

1. 配置 yarn-site.xml

   ```xml
       <!-- 开启日志聚集功能 -->
       <property>
           <name>yarn.log-aggregation-enable</name>
           <value>true</value>
       </property>
       <!-- 设置日志聚集服务器地址 -->
       <property>
           <name>yarn.log.server.url</name>
           <value>http://hadoop1:19888/jobhistory/logs</value>
       </property>
       <!-- 设置日志保留时间为7天 -->
       <property>
           <name>yarn.log-aggregation.retain-seconds</name>
           <value>604800</value>
       </property>
   ```

2. 分发配置

   ```shell
   scp /usr/local/hadoop-3.3.2/etc/hadoop/yarn-site.xml hadoop2:/usr/local/hadoop-3.3.2/etc/hadoop/yarn-site.xml
   scp /usr/local/hadoop-3.3.2/etc/hadoop/yarn-site.xml hadoop3:/usr/local/hadoop-3.3.2/etc/hadoop/yarn-site.xml
   ```

3. 重启NodeManager ResourceManager HistoryServer

   ```shell
   ssh hadoop2 'source /etc/profile && stop-yarn.sh'
   mapred --daemon stop historyserver
   
   ssh hadoop2 'source /etc/profile && start-yarn.sh'
   mapred --daemon start historyserver
   ```

4. 重新运行wordcount

   ```shell
   hadoop fs -rm -r -f /wcoutput
   hadoop jar /usr/local/hadoop-3.3.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.2.jar wordcount /wcinput /wcoutput
   
   # yarn查看 http://192.168.111.182:8088/
   ```

#### 3.2.7 各个组件启停总结

1. 各个模块分开启停

   ```shell
   # 整体启停HDFS
   start-dfs.sh
   stop-dfs.sh
   # 整体启停YARN
   start-yarn.sh
   stop-yarn.sh
   ```

2. 各个服务组件注意启停

   ```shell
   # 分别启停HDFS组件
   hdfs --daemon start/stop namenode/datanode/secondarynamenode
   # 分别启停YARN
   yarn --daemon start/stop resourcemanager/nodenamager
   ```

#### 3.2.8 常用端口号说明

| 端口名称                      | Hadoop 2.x | Hadoop 3.x     |
| ----------------------------- | ---------- | -------------- |
| NameNode 内部通信端口         | 8020/9000  | 8020/9000/9820 |
| NameNode HTTP UI (对用户暴露) | 50070      | 9870           |
| MapReduce 查看执行任务端口    | 8088       | 8088           |
| 历史服务器通信端口            | 19888      | 19888          |



## 4. 启动脚本

```
[hadoop@hadoop1 ~]$ cat Start.sh 
#!/bin/bash
start-dfs.sh
ssh hadoop2 'source /etc/profile && start-yarn.sh'
mapred --daemon start historyserver

[hadoop@hadoop1 ~]$ cat stop.sh 
#!/bin/bash
stop-dfs.sh
ssh hadoop2 'source /etc/profile && stop-yarn.sh'
mapred --daemon stop historyserver
```













