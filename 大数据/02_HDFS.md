## 1. 概述

### 1.1 HDFS的产生背景

- 随着数据量越来越大，在一个操作系统存不下所有数据，那么就分配到更多的操作系统管理的磁盘中，但是不方便管理和维护，迫切需要一种系统来管理多台机器上的文件，这就是分布式文件管理系统；

    HDFS只是分部署文件管理系统中的一种；

- HDFS（Hadoop Distributed File System），它是一个文件系统，用于存储文件，通过目录树来定位文件；

    其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色；

- HDFS的使用场景：适合一次写入，多次读出的场景；一个文件经过创建、写入和关闭之后就不需要改变。

### 1.2 优缺点

**优点**

- 高容错性

    数据自动保存多个副本；它通过增加副本的形式，提高容错性

    某一个副本丢失以后，它可以自动恢复

- 适合处理大数据

    数据规模：能够处理的数据规模达到GB、TB、甚至PB级别的数据

    文件规模：能够处理百万规模以上的文件数量

- 可以构建在廉价的机器上，通过多副本机制，提高可靠性

**缺点**

- 不适合低延时数据访问，比如毫秒级的存储数据，是做不到的

- 无法高效的对大量小文件进行存储

    存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息；

    小文件存储的寻址时间会超过读取时间，违反了HDFS的设计目标

- 不支持并发写入、文件随机修改

    一个文件只能有一个写，不允许多个线程同时写

    仅支持数据append（追加），不支持文件的随机修改

### 1.3 组成

- NameNode（nn）：就是Master，它是一个主管、管理者
    - 管理HDFS的名称空间
    - 配置服务策略
    - 管理数据库（Block）映射信息
    - 处理客户端读写请求
- DataNode：就是Slave；NameNode下达命令，DataNode执行实际操作
    - 存储实际的数据块
    - 执行数据块的读写操作
- Client：就是客户端
    - 文件切分；文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传
    - 与NameNode交互，获取文件的位置信息
    - 与DataNode交互，读取或者写入数据
    - Client提供一些命令来管理HDFS，比如NameNode的格式化
    - Client可以通过一些命令来访问HDFS，比如对HDFS增删改查操作
- SecondaryNameNode（2nn）：并非NameNode的热备；当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务
    - 复制NameNode，分担其工作量，比如定期合并Fsimage和Edits，并定期推送给NameNode
    - 在紧急情况下，可以辅助恢复NameNode

### 1.4 文件块大小

HDFS中的文件在物理上是分块存储（Block），块的大小可以通过配置参数（hdfs-default.xml -> dfs.blocksize）来规定，默认大小在Hadoop2.x/Hadoop3.x版本中是128M，1.x版本是64M



## 2. HDFS的Shell相关操作

**hadoop fs 等同于 hdfs dfs**

- **帮助:** -help

  ```shell
  hadoop fs -help rm
  ```

- **上传:** 

  - **-moveFromLocal**

    从本地剪切到HDFS

  - **-copyFromLocal**

    从本地拷贝到HDFS

  - **-put**

    等同于**-copyFromLocal**

  - **-appendTofile**

    追加一个文件到已存在文件的末尾

- **下载**

  - **-copyToLocal**

    从HDFS下载到本地

  - **-get**

    等同于**-copyToLocal**

- **HDFS直接操作**

  - **-ls**	显示目录信息

  - **-cat**	显示文件内容

  - **-chgrp -chmod -chown**	同linux命令, 修改文件所属权限

  - **-mkdir**	创建目录

  - **-cp**	从HDFS的一个路径拷贝到HDFS的另一个路径

  - **-mv**	在HDFS中移动文件

  - **-tail**	显示一个文件末尾1KB的数据

  - **-rm**	HDFS中删除文件或目录

  - **-rm -r**	递归删除目录及目录里面的内容

  - **-du**	统计文件夹的大小信息

  - **-setrep**	设置HDFS中文件的副本数量

    ```shell
    hadoop fs -setrep 10 /wcinput/aaa.txt
    # 这里设置的副本数据只是记录在NameNode的元数据中,是否真的会有这么多副本,还得看DataNode数量;
    # 目前只有3台设备, 最多也就3副本, 只有节点数增加到10台时,副本数才能达到10;
    ```



## 3. HDFS的客户端API

### 3.1 Windows环境设置

- Hadoop Client	https://github.com/cdarlint/winutils

- **环境变量**
  
  - **HADOOP**
    - **HADOOP_HOM**E C:\MySoft\hadoop-3.2.2
    - **PATH** %HADOOP_HOME%\bin
  - **MAVEN**
    - **MAVEN_HOME** C:\MySoft\apache-maven-3.6.3
    - **PATH** %MAVEN_HOME%\bin
- IDEA
  - 新建Maven工程 -> 下一步 -> GroupId: com.alec
  
    ​											  -> ArtifactId: HDFSClient
  
  - pom.xml
  
    ```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.3.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
        </dependency>
    </dependencies>
    ```
  
  - 右侧边栏 -> Maven -> 刷新按钮 -> 等待下载完成
  
  - 在 src/main/resource 创建 log4j.properties
  
    ```properties
    log4j.rootLogger=INFO, stdout
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
    log4j.appender.logfile=target/hdfs.log
    log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
    log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
    ```
  
  - 创建包名: com.alec.hdfs
  
  - 创建HdfsClient类
  
    ```java
    package com.alec.hdfs;
    
    public class HdfsClient {
        
    }
    ```

### 3.2 Java代码

客户端代码套路

1. 获取一个客户端对象
2. 执行相关的操作命令
3. 关闭资源

```java
package com.alec.hdfs;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;


public class HdfsClient {
    private FileSystem fs;

    @Before
    public void init() throws URISyntaxException, IOException, InterruptedException {
        // 连接的集群NameNode地址
        URI uri = new URI("hdfs://192.168.111.181:8020");
        // 创建一个配置文件
        Configuration configuration = new Configuration();
        // 指定连接用户
        String user = "hadoop";
        // 1. 获取到了客户端对象
        fs = FileSystem.get(uri, configuration, user);
    }

    // 创建目录
    @Test
    public void testMkdir() throws IOException {
        // 2. 执行命令
        fs.mkdirs(new Path("/xiyou/huaguoshan1"));
    }

    // 上传
    @Test
    public void testPut() throws IOException {
        // 参数解读: 参数1: 是否删除原始数据; 参数2: 是否允许覆盖; 参数3: 源数据路径; 参数4: 目的路径
        fs.copyFromLocalFile(false, false, new Path("D:\\sunwukong.txt"), new Path("/xiyou/huaguoshan"));
    }

    @After
    public void close() throws IOException {
        // 3. 关闭资源
        fs.close();
    }
}

```







## 4. HDFS的读写流程





## 5. NameNode和SecondaryNameNode







## 6. DataNode工作机制