[toc]

# 1. 安装
- 下载地址: https://mirrors.bfsu.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.zip
- 配置环境变量:
  - MAVEN_HOME=/usr/local/apache-maven-3.6.3
  - PATH=$PATH:$MAVEN_HOME/bin

# 2. Maven仓库

- 本地仓库
- 远程仓库(私有仓库)
- 中央仓库

配置本地仓库: conf/settings.xml
```xml
<localRepository>/data/maven_repository</localRepository>
```

配置代理

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```



# 3. Maven标准目录结构

- ProjectPath/src/main/java : 核心代码部分
- ProjectPath/src/main/resources : 配置文件部分
- ProjectPath/src/test/java : 测试代码部分
- ProjectPath/src/test/resources : 测试配置文件部分
- ProjectPath/src/main/webapp : web页面资源(js, css, 图片等)

# 4 Maven常用命令

进入项目目录 `ProjectPath`;

> mvn

- `mvn clean` 删除 target目录(编译后的字节码文件)
- `mvn compile` 编译核心代码, 保存到target/classes目录
- `mvn test` 编译核心代码和测试代码(target/test-classes)
- `mvn package` 编译 -> 构建war包(target/***.war), 在pom.xml中修改`<packaging>war</packaging>`
- `mvn install` 编译构建 并 保存到本地maven仓库

# 5. Maven生命周期

> 默认生命周期


- 编译 `mvn compile`
- 测试 `mvn test`
- 打包 `mvn package`
- 安装 `mvn install`
- 发布 `mvn deploy`


- 清理项目编译信息 `mvn clean`


> 站点生命周期(ignore)

# 6. 配置IDEA Maven插件
```
Setting -> Maven -> Maven home directory
                 -> User setting file
                 -> Local repository

Setting -> Maven -> Runner -> VM Options(-DarchetypeCatalog=internal)
```