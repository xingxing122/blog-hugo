---
title: "Jenkins 语法"
date: 2020-08-04T16:50:59+08:00
draft: false  
categories: [devops]
tags: [devops]

---

Jenkinsfile 

<!--more-->

Jenkinsfile 组成

```bash
     指定node节点/workspace 
     指定运行选项
     指定运行选项
     指定stages阶段
     指定构建后操作
```

#指定node 节点/workspace   

#agent

```bash 
pipeline {
    agent { node { label "master"  // 指定运行节点的标签或者名称   
                   customWorkspace "${workspace}"  //指定运行工作目录(可选)
      }
  }
options{
    timestamps()   // 日志会有时间
    skipDefaultCheckout() //删除模式checkout  scm 语句
    disableConcurrentBuilds() // 禁止并行
    timeout(time: 1,uint: 'HOURS') //流水线超时设置1h
}    
```
#Pipeline 定义-stage  

```bash
指定stages 阶段(一个或多个)
模板添加了三个阶段
* GetCode 
* Build 
* CodeScan 

 stage {
    //下载代码
    stage ("GetCode") { // 阶段名称
       steps {  //步骤
         timeout(time:5,uint: "MINUTES") {   //步骤超时时间
             script{  // 填写运行代码
                println(‘获取代码’)
             }
         }
       }
    }
 }
 
//构建
stage("Build") {
   steps{
       timeout(time:20,uint:"MINUTES"){
          script{
             println('应用打包')
          }
        }   
      }
   }

//代码扫描
stage("CodeScan"){
   steps{
      timeout(time:30, unit:"MINUTES"){
        script{
         print("代码扫描")
        }
      }
   } 
 }
```

#Pipeline 定义-post 

 指定构建后操作

解释:

```bash
always{}:  总是执行脚本片段
success{}: 成功后执行
failure{}: 失败后执行
aborted{}: 取消后执行

currendBuild 是一个全局变量
  description  构建描述
```

```bash
//构建后操作
post{
   always {
    script{
      println("always") 
     }
   }
 }
success{
   script{
    currentBuild.description += "\n 构建成功!"
    }
 }
 failure {
     script{
       currentBuild.description   += "\n构建失败"   
     }
 }
 aborted {
     script {
         currentBuild.description  +="\n 构建取消"
     }
 }
```

Pipeline  语法-agent  

```bash
agent (代理)
  agent 指定了流水线的执行节点
参数: 
   any  在任何可用的节点上执行pipeline
   node 没有指定agent的时候默认
   label 指定标签上的节点运行Pipeline 
   node  允许额外的选项
   
   这两种是一样的
   agent {node {label 'labelname'}}
   agent { label 'labelname'}   
```

```bash
post 
定义一个或多个steps,这些阶段根据流水线的完成情况而运行(取决于post部分的位置)，post支持以下post-condition块中其中之一；always,changed,failure,success,unstable和aborted. 这些条件块允许在post 部分的步骤执行取决于流水线或阶段的完成状态

   always  无论流水线或者阶段的完成状态
   changed 只有当流水线或者阶段完成状态与之前不同时 
   failure 只有当流水线或者阶段状态为"failure"运行
   success 只有当流水线或者阶段状态为"success"运行
   unstable 只有当流水线或者阶段状态为"unstable"运行，如测试失败
   aborted 只有当流水线或者阶段状态为"aborted"运行， 手动取消
   
```

```bash
stages(阶段)
 包含一系列一个或多个stage指令，建议stages至少包含一个stage 指定用于连续交付过程中的每个离散部分，比如构建 测试  和部署
 pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

```bash
steps (步骤)
step 是每个阶段中要执行的每个步骤
 pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

参数

![image-20200828172559817](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-28-092600.png)



trigger 触发器 

```bash
triggers { cron ( 'H */4  * *  1-5  ')}
```

tool 

获取通过自动安装或手动放置工具的环境变量。支持maven/jdk/gradle，工具的名称必须在系统设置-全局工具配置中定义  

when 

when 指令允许流水线根据给定的条件决定是否应该执行阶段。 when指令必须包含至少一个条件。如果when 指令包含多个条件，所有的条件必须返回true，阶段才能执行 

```bash
when {branch  'master' }
```

script 

```bash
script 步骤需要[scripted-pipline ] 块并在声明式中执行。对于大多数用例来说，应该声明式流水线中的"脚本"步骤是不必要的，但是它可以提供一个有用的"逃生出口"，非平凡的规模和/或复杂性的script 块应该被转移到共享库
```

















