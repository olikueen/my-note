[toc]

# 1. 流水线基本语法

## 1.1 Pipeline定义

- 一条流水线通过Jenkinsfile描述
- 安装声明式插件Pipeline: Declarative
- Jenkinsfile组成
  - 指定node节点 / workspace
  - 指定运行选项
  - 指定stages阶段
  - 指定构建后操作

**声明式 Jenkinsfile Demo**

```groovy
pipeline{
    agent{
        node{
            label "mater" // 指定运行节点的标签或名称
            customWorkspace "${workspace}" // 指定运行工作目录(可选)
        }
    }
    options{
        timestamps() // 给日志添加时间, 需要安装Timestamper插件
        skipDefaultCheckoput() // 删除隐式 checkout scm 语句
        disableConcurrentBuilds() // 禁止并行
        timeout(time: 1, unit: "HOURS") // 流水线超时时间
    }
    stages {
        //下载代码
        stage("PullCode"){ // 阶段名称
            steps { // 步骤
                timeout(time: 5, unit: "MINUTES"){ // 步骤超时时间
                    script { // 编写运行代码
                        println("获取代码")
                    }
                }
            }
        }
        // 构建
        stage("Build"){
            steps{
                timeout(time: 5, unit: "MINUTES"){
                    script{
                        println("应用构建")
                    }
                }
            }
        }
        // 代码扫描
        stge("CodeScan"){
            steps{
                timeout(time: 5, unit: "MINUTES"){
                    script{
                        println("代码扫描")
                    }
                }
            }
        }
    }
    // 构建后操作
    post{
        always{ // 每次构建后总是执行
            script{ println("always") }
        }
        success{
            script{ currnetBuild.description += "\n 构建成功" }
        }
        failure{
            script{ currnetBuild.description += "\n 构建失败" }
        }
        aborted{
            script{ currnetBuild.description += "\n 构建取消" }
        }
    }
}
```

### 1.1.1 agent/options

- 指定node节点 / workspace
- 指定运行选项(可以省略)

```groovy
agent{
    node{
        label "mater" // 指定运行节点的标签或名称
        customWorkspace "${workspace}" // 指定运行工作目录(可选)
    }
}
options{
    timestamps() // 给日志添加时间, 需要安装Timestamper插件
    skipDefaultCheckoput() // 删除隐式 checkout scm 语句
    disableConcurrentBuilds() // 禁止并行
    timeout(time: 1, unit: "HOURS") // 流水线超时时间
}
```

### 1.1.2 stages

- 指定stage阶段(一个或多个)
- 解释: Demo中有三个阶段

```groovy
stages {
    //下载代码
    stage("PullCode"){ // 阶段名称
        steps { // 步骤
            timeout(time: 5, unit: "MINUTES"){ // 步骤超时时间
                script { // 编写运行代码
                    println("获取代码")
                }
            }
        }
    }
    // 构建
    stage("Build"){
        steps{
            timeout(time: 5, unit: "MINUTES"){
                script{
                    println("应用构建")
                }
            }
        }
    }
    // 代码扫描
    stge("CodeScan"){
        steps{
            timeout(time: 5, unit: "MINUTES"){
                script{
                    println("代码扫描")
                }
            }
        }
    }
}
```

### 1.1.3 post

- 指定构建后的操作
- 解释:
  - always{} 总是在构建执行的脚本片段
  - success{} 构建成功后执行
  - failure{} 构建失败后执行
  - aborted{} 取消后执行
- currentBuild 是一个全局变量
  - description: 构建描述

```groovy
// 构建后操作
post{
    always{ // 每次构建后总是执行
        script{ println("always") }
    }
    success{
        script{ currnetBuild.description += "\n 构建成功" }
    }
    failure{
        script{ currnetBuild.description += "\n 构建失败" }
    }
    aborted{
        script{ currnetBuild.description += "\n 构建取消" }
    }
}
```

## 1.2 Pipeline语法

### 1.2.1 agent

agent 指定了流水线的执行节点

**参数:**

- any 在任何可用的节点上执行pipeline
- none没有指定anent的时候默认
- label在指定标签上的节点允许pipeline
- node 允许额外的选项(如: customWorkspace)

```groovy
// 这两种是一样的
agent { node { label 'label name' }}
agent { label ' label name' }
```

### 1.2.2 post

定义一个或多个steps, 这些阶段根据流水线或者阶段的完成情况而运行(取决于流水线中post部分的位置)

post支持以下post-condition块: always, changed, failure, success, unstable, aborted

这些条件块允许 在post部分的步骤执行 取决于流水线或者阶段的完成状态

- always 无论流水线或者阶段的完成状态
- changed 只有当流水线或者阶段的完成状态与之前不同时
- failure 只有当流水线或者阶段状态为"failure"时运行
- success 只有当流水线或者阶段状态为"success"时运行
- unstable 只有当流水线或者阶段状态为"unstable"时运行; 例如: 测试失败
- aborted 只有当流水线或者阶段状态为"aborted"时运行; 例如: 手动取消

### 1.2.3 stages

阶段

包含一系列一个或者多个stage指令, 建议stages至少包含一个stage指令用于连续交付过程的每个离散部分, 比如构建, 测试, 和部署

### 1.2.4 steps

步骤

steps是每个阶段中要执行的每个步骤

### 1.2.5 指令

#### environment

环境变量

environment 指令指定一个键值对序列, 该序列将被定义为所有步骤的环境变量, 或者用于特定阶段的步骤; 这取决于 environment 指令在流水线内的位置;

该指令支持一个特殊的方法 credentials(), 该方法可以用于 Jenkins 环境中通过标识访问预定义的凭证; 对于类型为 "Secret Text" 的凭证, credentials() 将确保指定的环境变量包含秘密文本内容;

对于类型为 "Standard username and password" 的凭证, 指定的环境变量指定为 `username:password`, 并且两个额外的环境变量将被自动定义: 分别为 MYVARNAME_USR 和 MYVARNAME_PSW;

```groovy
pipeline {
    agent any
    environment {
        CC = 'clang'
    }
    stages {
        stage("Example") {
            evironment{
                AN_ACCESS_KEY = credentials('my-prefined-secret-text')
            }
            steps{
                sh 'print env'
            }
        }
    }
}
```

#### options

options 指令允许从流水线内部配置特定于流水线的选项; 流水线提供了许多这样的选项, 比如`buildDiscarder`, 但也可以由插件提供, 比如 timestamps;

- buildDiscarder: 为最近的流水线运行的特定数量保存组件和控制台输出
- disableConcurrentBuilds: 不允许同时执行流水线; 可被用来防止同时访问共享资源等;
- overrideIndexTriggers: 允许覆盖分支索引触发器的默认处理;
- skipDefaultCheckout: 在`agent`指令中, 跳过从源代码控制中检出代码的默认情况;
- skipStagesAfterUnstable: 一旦构建状态变得UNSTABLE, 跳过该阶段;
- checkoutToSubdiretory: 在工作空间的子目录中自动执行源代码控制检出;
- timeout: 设置流水线运行的超时时间, 在此之后, Jenkins将中止流水线;
- retry: 在失败时, 重新尝试整个流水线的指定次数;
- timestamps: 预测所有由流水线生成的控制台输出, 与该流水线发出的时间一致;

#### parameters

为流水线运行时设置项目相关的参数

- string 字符串类型的参数

  ```groovy
  parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }
  ```

- booleanParam 布尔参数

  ```groovy
  parameters { booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description:'') }
  ```

#### triggers

构建触发器

- cron 计划任务定期执行构建;

  ```groovy
  triggers { cron('H */4 * * 1-5') }
  ```

- pollSCM 与 cron 定义类似, 但是由 jenkins 定期检测源码变化;

  ```groovy
  triggers { pollSCM('H */4 * * 1-5') }
  ```

- upstream 接受逗号分隔的工作字符串和阈值; 当字符串中的任何作业以最小阈值结束时, 流水线被重新触发;

  ```groovy
  triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }
  ```

#### tools

获取通过自动安装或手动放置工具的环境变量; 支持 maven/jdk/gradel; 工具的名称必须在系统设置 -> 全局工具配置中定义;

```groovy
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1'
    }
    stages {
        stage("Example") {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

#### input

input 用户在执行各个阶段的时候, 由人工确认是否继续进行(处理交互)

- message 呈现给用户的提示信息
- id 可选, 默认为 stage 名称
- ok 默认表单上的 ok 文本
- submitter 可选, 以逗号分隔的用户类别或允许提交的外部组名; 默认允许任何用户
- submitterParameter 环境变量的可选名称; 如果存在, 用 submitter 名称设置
- parameters 提示提交者提供一个可选的参数列表

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            input {
                message "Should we continue?"
                ok "Yes, we should."
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: ' Mr Jenkins', description: 'Who should I say hello?')
                }
            }
        }
        steps {
            echo "Hello, ${PERSON}, nice to meet you."
        }
    }
}
```

#### when

when 指令 允许流水线根据给定的条件决定是否应该执行阶段; when 指令必须包含至少一个条件;

如果 when 指令包含多个条件, 所有的子条件必须返回 True, 阶段才能继续执行;

这与子条件在 allOf 条件下嵌套的情况相同;

内置条件

- branch: 当正在构建的分支与模式给定的分支匹配时, 执行这个stage, 这只适用于多分支流水线

  ```groovy
  when { branch 'master' }
  ```

- environment: 当指定的环境变量是给定的值时, 执行这个stage

  ```groovy
  when { environment name: 'DEPLOY_TO', value: 'production' }
  ```

- expression 当指定的 Groovy 表达式评估为 true 时, 执行这个stage

  ```groovy
  when { expression { return paras.DEBUG_BUILD }}
  ```

- not 当嵌套的条件是错误时, 执行这个stage, 必须包含一个条件

  ```groovy
  when { not { branch 'master' }}
  ```

- allOf 当所有的嵌套条件都正确时, 执行这个stage, 必须包含一个条件

  ```groovy
  when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' }}
  ```

- anyOf 当至少有一个嵌套条件为真时, 执行这个阶段, 必须包含至少一个条件

  ```groovy
  when { anyOf { branch 'master'; branch 'staging' }}
  ```

```groovy
pipeline {
    agent any
    stages {
        stage('example') {
	        when { environment name: 'test', value: 'abcd' }
            steps {
                echo 'example'
            }
        }
    }
}
```

#### parallel

并行 声明式流水线的阶段可以在他们内部声明多隔嵌套阶段, 它们将并行执行; 

注意, 一个阶段必须只有一个 steps 或 parallel 阶段;

嵌套阶段本身不能包含 进一步的 pallel 阶段, 但是其他的阶段的行为与任何其他 stage parallel 的阶段不能包含 agent 或 tools 阶段, 因为它们没有相关 steps;

另外, 通过添加 failFast true 到包含 parallel 的 stage 中, 当其中一个进程失败时, 你可以强制所有的 parallel 阶段都被中止;

```groovy
stages {
    stage('example') {
        when { branch 'master' }
    }
    failFast true
    parallel {
        stage('Branch A') {
            agent { label 'for branch A' }
            steps {
                echo 'On branch A'
            }
        }
        stage('Branch B') {
            agent { label 'for branch B' }
            steps {
                echo 'On branch B'
            }
        }
    }
}
```

### 1.2.6 step 步骤

#### script

script 步骤需要 [scripted-pipeline] 块并在声明式流水线中执行; 对于大多数用例来说, 应该声明式流水线中的"脚本"步骤是不必要的, 但是它可以提供一个有用的"逃生出口"; 非平凡的规模和/或复杂性的script块 应该被转移到 共享库;

```groovy
pipeline {
    agent any
    stages {
        stage('example') {
            echo "Hello world"
            script {
                def browsers = ['chrome', 'firefox']
                for (int i=0; i< browsers.size(); i++) {
                    echo "Testing the ${browsers[i]} browser"
                }
            }
        }
    }
}
```

## 1.3 Share Library

- src 目录类似于标准 Java 源目录结构; 执行流水线是, 此目录将添加到类路径中;
- vars 目录托管脚本文件, 这些脚本在"管道"中作为变量公开;
- resources 目录允许 libraryResource 从外部库中使用步骤来加载相关联的非 groovy 文件;

![image-20221130232811741](C:\my-note\CICD\media\image-20221130232811741.png)

### 1.3.1 sharelibrary 的使用

- **在 Jenkins Server 安装 git**

- **Github创建一个public仓库 jenkins-sharelib**

   ![image-20221130235317735](.\media\image-20221130235317735.png)

  > tools.groovy

  ```goroovy
  package org.devops
  
  //打印内容
  def PrintMsg(content) {
      println(content)
  }
  ```

  > hello.groovy

  ```groovy
  def call() {
      println("hello")
  }
  ```

- **配置Sharelibrary:** Jenkins -> Manage Jenkins -> Configure System -> Global Pipeline Libraries -> 新增

  ![image-20221130235916604](.\media\image-20221130235916604.png)

- **调用sharelibrary**

  - 在item中配置pipeline

    ```groovy
    @Library('jenkins-sharelib@master') _
    def tools = new org.devops.tools()
    pipeline{
        agent { node { label 'build' }}
        stages{
            stage("example"){
                steps{
                    println("获取代码")
                    script {
                        sh 'hostname'
                        tools.PrintMsg("hello sharelib")
                        hello()
                    }
                }
            }
        }
    }
    ```

    

  - 使用Jenkinsfile

     ![image-20221201003058415](C:\my-note\CICD\media\image-20221201003058415.png)

    在item中配置Jenkinsfile

    ![image-20221201003226878](C:\my-note\CICD\media\image-20221201003226878.png)

### 1.3.2 实践: 日志颜色输出

- 安装插件 AnsiColor 插件

- 更新代码

  > tools.grovy

  ```groovy
  package org.devops
  
  //打印内容
  def PrintMsg(content, color) {
      println("hello sharelib")
      def colors = ["red": "\033[40;31m >>>>>>>>>> ${content} <<<<<<<<<< \033[0m",
                "blue": "\033[47;34m >>>>>>>>>> ${content} <<<<<<<<<< \033[0m",
                "green": "\033[40;32m >>>>>>>>>> ${content} <<<<<<<<<< \033[0m" ]
      ansiColor('xterm') {
          println(colors[color])
      }
  }
  ```

  > Jenkinsfile

  ```groovy
  @Library('jenkins-sharelib@master') _
  def tools = new org.devops.tools()
  pipeline{
      agent { node { label 'build' }}
      options {
          timestamps()
          skipDefaultCheckout()
      }
      stages{
          stage("example"){
              steps{
                  println("获取代码")
                  script {
                      sh 'hostname'
                      tools.PrintMsg("hello color sharelib", "red")
                      tools.PrintMsg("hello color sharelib", "blue")
                      tools.PrintMsg("hello color sharelib", "green")
                      hello()
                  }
              }
          }
      }
  }
  ```

  

  



