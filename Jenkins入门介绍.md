# Jenkins入门介绍

# 什么是Jenkins？

Jenkins是完整的开源自动化任务服务器，能够构建、测试、部署软件。

```
java -jar jenkins.war --httpPort=8080
```

初次登录的时候需要填写Administrator password，相应信息记录在:
```
/root/.jenkins/secrets/initialAdminPassword
```

然后需要配置代理，参考[cntlm代理配置](#cntlm代理配置)。

jenkins官方的文档其实比较少，但是并不妨碍我们使用jenkins这个工具，因为软件本身就是自解释的，举例：[hello](http://100.120.252.146:8080/job/hello/)。

# Jenkins Pipeline

Jenkins Pipeline 是一套可以集成到jenkins中的运输管道插件，并且提供了一套工具来将管道运输通过类似代码的方式建模。

最简单的pipeline如下：
```
node {
   echo 'Hello World'
}
```
**node**：allocates an executor and workspace in the Jenkins environment.

Jenkins Pipeline 可以通过webUI或者Jenkinsfile[两种方式](http://100.120.252.146:8080)定义，Jenkinsfile被认为是一种最好的方式，可以随源代码管理。

```
pipeline {
    agent { docker 'maven:3.3.3' }
    stages {
        stage('build') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

举例参考:[Creating your first Pipeline](https://jenkins.io/doc/pipeline/tour/hello-world/#examples)

Pipelines由多个step构成，每个step就像一个命令，当一个step执行成功后，就会移动到下一个step，如果一个step失败，那么整个Pipeline都将失败，所有的step都执行成功，才认为Pipeline执行成功。

除了一些常见的命令式step外，Jenkins还提供了一个包装型step，比如timeout和retry

```
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                retry(3) {
                    sh './flakey-deploy.sh'
                }

                timeout(time: 3, unit: 'MINUTES') {
                    sh './health-check.sh'
                }
            }
        }
    }
}
```

包装step还可以包含包装step：

```
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    retry(5) {
                        sh './flakey-deploy.sh'
                    }
                }
            }
        }
    }
}
```

通常Pipeline执行完之后还需要进行一些处理收尾工作，在post中实现：

```
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'echo "Fail!"; exit 1'
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
```

## 定义执行环境

如下，[agent](https://jenkins.io/doc/book/pipeline/syntax#agent)就是用来定义执行环境的：
```
pipeline {
    agent {
        docker { image 'node:7-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```

# Jenkins管理

## 插件管理

- 通用插件：pipeline，git　
- 私有插件：sourceMonitor等

## 节点管理
[Jenkins使用教程之管理节点](http://www.jianshu.com/p/047362b11403)

# 其他

## cntlm代理配置

Jenkins例子中需要使用docker，关于docker的安装可以参考[计算云Docker安装说明](http://3ms.huawei.com/hi/group/8319/wiki_4509543.html)，其中涉及到cntlm代理配置。

简要步骤记录如下：
1. 安装工具cntlm：二级代理，相当于应用将网络请求发送给cntlm（127.0.0.1:3128），cntlm发送给server。
   ```
   apt-get install cntlm
   ```
2. 修改配置  
   配置文件：/etc/cntlm.conf
   ```
   Username        z00123456  (域账户名称)
   Domain          china
   Password
   Proxy           proxyhk.huawei.com:8080
   NoProxy         localhost, 127.0.0.*, 10.*, 192.168.*, 100.100.*
   Listen          3128
   # 下面两个配置原理上应该可以不用配置，目前已经配置，暂未测试
   Gateway         yes
   Allow           127.0.0.1
   ```
3. 生成鉴权码
   ```
   cntlm -c /etc/cntlm.conf -M http://www.google.com
   ----------------------------[ Profile  1 ]------
   Auth            NTLM
   PassNT          1CAFFAEB18D5295C93E4538574E8CDA8
   PassLM          4A0E1387F4FA86864B2482A5E4AB4967
   ------------------------------------------------
   ```
   将生成的信息覆盖/etc/cntlm.conf中相应行。
4. 重启服务
   ```
   service cntlm restart
   ```
5. 设置应用代理
  - 对于shell
    在~/.bashrc中加入下面代码
    ```
    export http_proxy=http://127.0.0.1:3128
    export https_proxy=http://127.0.0.1:3128
    ```
    然后执行验证代理配置是否正常
    ```
    source ./bashrc
    curl www.baidu.com
    ```
  - 对于docker
    ```

    ```
  - 对于apt-key
    ```
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --keyserver-options http-proxy=http://localhost:3128 --recv 0C49F3730359A14518585931BC711F9BA15703C6
    ```

## java环境安装（jdk1.8+）

1. Oracle官网下载对应的[安装包](http://www.oracle.com/technetwork/cn/java/archive-139210-zhs.html)  
   我下载的是jdk-8u144-linux-x64.tar.gz
2. 解压缩后放到指定目录
   ```bash
   mkdir /usr/lib/jvm
   tar -zxvf jdk-8u144-linux-x64.gz -C /usr/lib/jvm
   ```
3. 配置bashrc
   在~/.bashrc中加入下面内容：
   ```
   #set oracle jdk environment
   export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_144  ## 这里要注意目录要换成自己解压  jdk  目录
   export JRE_HOME=${JAVA_HOME}/jre 
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
   export PATH=${JAVA_HOME}/bin:$PATH
   ```
   生效：
   ```
   source ~/.bashrc
   ```
4. 如果已经安装了其他版本的jdk，则需要指定默认jdk版本，具体参考：[Ubuntu下安装jdk8](http://3ms.huawei.com/km/blogs/details/2274377?l=zh-cn)


## FAQ

[jenkins pipeline: agent vs node?](https://stackoverflow.com/questions/42050626/jenkins-pipeline-agent-vs-node)
