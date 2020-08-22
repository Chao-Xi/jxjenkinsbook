# 第二节 基于Docker的pipeline流水线


## 1、容器中编译流水线作业

### 1-1 准备工作


配置`JenkinsMaster`挂载Docker

```
docker run -d -p 8080:8080 -p 50000:50000 --env=JAVA_OPTS=-Djenkins.install.runSetupWizard=false -v /var/lib/jenkins:/var/jenkins_home  -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker jenkins:v20200816
```

解决权限问题/以`root`用户运行


```
docker exec -it 005a7714ecc9 bash
jenkins@005a7714ecc9:/$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```

```
$ docker exec -it -u root 005a7714ecc9 bash

root@005a7714ecc9:/# docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                              NAMES
005a7714ecc9        jenkins:v20200816       "/sbin/tini -- /usr/…"   4 minutes ago       Up 4 minutes        0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   hopeful_pascal
384516ab8579        jenkins/inbound-agent   "jenkins-agent -url …"   12 minutes ago      Up 12 minutes                                                          blissful_nash


root@005a7714ecc9:/# usermod -aG root jenkins
root@005a7714ecc9:/# exit
```

```
$ docker restart 005a7714ecc9
$ id jenkins
uid=1000(jenkins) gid=1000(jenkins) groups=1000(jenkins),0(root)
```

**我的问题**

我在主机上创建了`docker`的`group`, 根据这个文章 https://docs.docker.com/engine/install/linux-postinstall/

```
sudo groupadd docker
sudo usermod -aG docker $USER

$ cat /etc/group
...
docker:x:993:vagrant


$ docker exec -it -u root 005a7714ecc9 bash
$ groupadd -g 993 docker-test

root@005a7714ecc9:/# usermod -aG docker-test jenkins
root@005a7714ecc9:/# id jenkins
uid=1000(jenkins) gid=1000(jenkins) groups=1000(jenkins),0(root),1001(docker),993(docker-test)

root@005a7714ecc9:/# cat /etc/group
...
docker-test:x:993:jenkins

$ docker exec -it 005a7714ecc9 bash
jenkins@005a7714ecc9:/$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                              NAMES
005a7714ecc9        jenkins:v20200816   "/sbin/tini -- /usr/…"   5 hours ago         Up 3 hours          0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   hopeful_pascal
```






jenkins should have permission now.

[Getting “Permission Denied” error when pulling a docker image in Jenkins docker container on Mac](https://medium.com/swlh/getting-permission-denied-error-when-pulling-a-docker-image-in-jenkins-docker-container-on-mac-b335af02ebca)


### 1-2 测试流水线

```
pipeline {
    agent {
        docker { 
            image 'maven:3.6.3-jdk-8' 
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -v'
            }
        }
    }
}
```

**Console Output**

```
Running on Jenkins in /var/jenkins_home/workspace/docker-pipeline1
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ docker inspect -f . maven:3.6.3-jdk-8
.
[Pipeline] withDockerContainer
Jenkins seems to be running inside container 005a7714ecc980f673c3083030ff79a82687681de55d9cd9875625a5aa0520f3
$ docker run -t -d -u 1000:1000 -v $HOME/.m2:/root/.m2 -w /var/jenkins_home/workspace/docker-pipeline1 --volumes-from 005a7714ecc980f673c3083030ff79a82687681de55d9cd9875625a5aa0520f3 -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** maven:3.6.3-jdk-8 cat
$ docker top c8aec18483a36a66c120975c09b667ba0292a7abebbe3b4409a5bbc1d23469ae -eo pid,comm
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] sh
+ mvn -v
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/share/maven
Java version: 1.8.0_265, vendor: Oracle Corporation, runtime: /usr/local/openjdk-8/jre
Default locale: en, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.12.2.el7.x86_64", arch: "amd64", family: "unix"
[Pipeline] sleep
Sleeping for 30 sec
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
$ docker stop --time=1 c8aec18483a36a66c120975c09b667ba0292a7abebbe3b4409a5bbc1d23469ae
$ docker rm -f c8aec18483a36a66c120975c09b667ba0292a7abebbe3b4409a5bbc1d23469ae
[Pipeline] // withDockerContainer
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```


### 1-3 前后端未分离项目

对于代码库中既包含前端项目又包含后端项目的配置。可以启动多个容器。

```
pipeline {
    agent none
    stages {
        stage('ServiceBuild') {
            agent {
                docker { 
                    image 'maven:3.6.3-jdk-8' 
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn -v  && sleep 15'
            }
        }
      
        stage('WebBuild') {
            agent {
                docker { 
                    image 'node:7-alpine' 
                    args '-v $HOME/.npm:/root/.npm'
                }
            }
            steps {
                sh 'node -v  && sleep 15'
            }
        }
    }
}
```

**Console output**

```
Running on Jenkins in /var/jenkins_home/workspace/docker-pipeline2
[Pipeline] {
[Pipeline] isUnix (hide)
[Pipeline] sh
+ docker inspect -f . maven:3.6.3-jdk-8
.
[Pipeline] withDockerContainer
Jenkins seems to be running inside container 005a7714ecc980f673c3083030ff79a82687681de55d9cd9875625a5aa0520f3
$ docker run -t -d -u 1000:1000 -v $HOME/.m2:/root/.m2 -w /var/jenkins_home/workspace/docker-pipeline2 --volumes-from 005a7714ecc980f673c3083030ff79a82687681de55d9cd9875625a5aa0520f3 -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** maven:3.6.3-jdk-8 cat
$ docker top 9f15b56640e3d65660876a4f854918103f7f099150dbb5f99dff43ace1296392 -eo pid,comm
[Pipeline] {
[Pipeline] sh
+ mvn -v
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/share/maven
Java version: 1.8.0_265, vendor: Oracle Corporation, runtime: /usr/local/openjdk-8/jre
Default locale: en, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.12.2.el7.x86_64", arch: "amd64", family: "unix"
+ sleep 15
[Pipeline] }
$ docker stop --time=1 9f15b56640e3d65660876a4f854918103f7f099150dbb5f99dff43ace1296392
$ docker rm -f 9f15b56640e3d65660876a4f854918103f7f099150dbb5f99dff43ace1296392
[Pipeline] // withDockerContainer
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (WebBuild)
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/docker-pipeline2
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ docker inspect -f . node:7-alpine
.
[Pipeline] withDockerContainer
Jenkins seems to be running inside container 005a7714ecc980f673c3083030ff79a82687681de55d9cd9875625a5aa0520f3
$ docker run -t -d -u 1000:1000 -v $HOME/.npm:/root/.npm -w /var/jenkins_home/workspace/docker-pipeline2 --volumes-from 005a7714ecc980f673c3083030ff79a82687681de55d9cd9875625a5aa0520f3 -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** node:7-alpine cat
$ docker top 33be7fc88b0f15f3cd3f947250e3b0d6d57f05096b6bac2bee7006cf0e380680 -eo pid,comm
[Pipeline] {
[Pipeline] sh
+ node -v
v7.10.1
+ sleep 15
[Pipeline] }
$ docker stop --time=1 33be7fc88b0f15f3cd3f947250e3b0d6d57f05096b6bac2bee7006cf0e380680
$ docker rm -f 33be7fc88b0f15f3cd3f947250e3b0d6d57f05096b6bac2bee7006cf0e380680
[Pipeline] // withDockerContainer
```


### 1-4 前端项目流水线

```
$ cd /home/vagrant/workspace/workspace/demo-pipeline3

// 安装 vue-cli

$ sudo chown -R 1000:1000 "/home/vagrant/.npm"


$  npm install -g @vue/cli-init
npm WARN deprecated vue-cli@2.9.6: This package has been deprecated in favour of @vue/cli
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
npm WARN deprecated coffee-script@1.12.7: CoffeeScript on NPM has moved to "coffeescript" (no hyphen)
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
+ @vue/cli-init@4.5.4
added 251 packages from 206 contributors in 17.774s

$ vue --version
@vue/cli 4.5.4

$ cd /home/vagrant/workspace/workspace/demo-pipeline3
$ $ vue init webpack demo

? Project name demo
? Project description A Vue.js project
? Author 
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Set up unit tests Yes
? Pick a test runner jest
? Setup e2e tests with Nightwatch? No
? Should we run `npm install` for you after the project has been created? (recommended) npm

   vue-cli · Generated "demo".


# Installing project dependencies ...
```

编写jenkinsfile

```
pipeline {
   agent {node {label "hostmachine"}}
    stages {
        stage('WebBuild') {
            steps {
                script {
                    docker.image('node:10.19.0-alpine').inside('-u 0:0 -v /var/jenkins_home/.npm:/root/.npm') {
                
                
                        sh """
                            id 
                            ls /root/.npm

                            ls /root/ -a
                            npm config set unsafe-perm=true
                            npm config list
                            npm config set cache  /root/.npm
                            #npm config set registry https://registry.npm.taobao.org
                            npm config list
                            ls 
                            cd demo && npm install  --unsafe-perm=true && npm run build  && ls -l dist/ && sleep 15 
                        """
                    }
                }
            }
        }
    }
}
```

**Console output**

```
$ docker run -t -d -u 1000:1000 -u 0:0 -v /var/jenkins_home/.npm:/root/.npm -w /home/vagrant/workspace/workspace/demo-pipeline3 -v /home/vagrant/workspace/workspace/demo-pipeline3:/home/vagrant/workspace/workspace/demo-pipeline3:rw,z -v /home/vagrant/workspace/workspace/demo-pipeline3@tmp:/home/vagrant/workspace/workspace/demo-pipeline3@tmp:rw,z -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** node:10.19.0-alpine cat
$ docker top ea767dafdc3ba98baf9399ebcde3fd491911d77a0c993f92a8a3b6f44a51fc7a -eo pid,comm
[Pipeline] {
[Pipeline] sh
+ id
uid=0(root) gid=0(root)
+ ls /root/.npm
_cacache
_locks
_logs
anonymous-cli-metrics.json
+ npm config set 'unsafe-perm=true'
+ npm config list
; cli configs
metrics-registry = "https://registry.npmjs.org/"
scope = ""
user-agent = "npm/6.13.4 node/v10.19.0 linux x64 ci/jenkins"

; userconfig /root/.npmrc
unsafe-perm = true

; node bin location = /usr/local/bin/node
; cwd = /home/vagrant/workspace/workspace/demo-pipeline3
; HOME = /root
; "npm config ls -l" to show all defaults.

+ npm config set cache /root/.npm
+ npm config list
; cli configs
metrics-registry = "https://registry.npmjs.org/"
scope = ""
user-agent = "npm/6.13.4 node/v10.19.0 linux x64 ci/jenkins"

; userconfig /root/.npmrc
cache = "/root/.npm"
unsafe-perm = true

; node bin location = /usr/local/bin/node
; cwd = /home/vagrant/workspace/workspace/demo-pipeline3
; HOME = /root
; "npm config ls -l" to show all defaults.

+ ls
demo
package-lock.json
+ cd demo
+ npm install '--unsafe-perm=true'
Cannot contact hostmachine: java.lang.InterruptedException
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.1.3 (node_modules/chokidar/node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.13 (node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.13: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

audited 1734 packages in 10.906s

28 packages are looking for funding
  run `npm fund` for details

found 89 vulnerabilities (73 low, 9 moderate, 7 high)
  run `npm audit fix` to fix them, or `npm audit` for details
+ npm run build

> demo@1.0.0 build /home/vagrant/workspace/workspace/demo-pipeline3/demo
> node build/build.js

Hash: [1mc4e4fdef7b4fbb5efe81[39m[22m
Version: webpack [1m3.12.0[39m[22m
Time: [1m12173[39m[22mms
                                                  [1mAsset[39m[22m       [1mSize[39m[22m  [1mChunks[39m[22m  [1m[39m[22m           [1m[39m[22m[1mChunk Names[39m[22m
               [1m[32mstatic/js/vendor.936b7041a764ab1c3f2c.js[39m[22m     123 kB       [1m0[39m[22m  [1m[32m[emitted][39m[22m  vendor
                  [1m[32mstatic/js/app.b22ce679862c47a75225.js[39m[22m    11.6 kB       [1m1[39m[22m  [1m[32m[emitted][39m[22m  app
             [1m[32mstatic/js/manifest.2ae2e69a05c33dfc65f8.js[39m[22m  857 bytes       [1m2[39m[22m  [1m[32m[emitted][39m[22m  manifest
    [1m[32mstatic/css/app.30790115300ab27614ce176899523b62.css[39m[22m  432 bytes       [1m1[39m[22m  [1m[32m[emitted][39m[22m  app
[1m[32mstatic/css/app.30790115300ab27614ce176899523b62.css.map[39m[22m  797 bytes        [1m[39m[22m  [1m[32m[emitted][39m[22m  
           [1m[32mstatic/js/vendor.936b7041a764ab1c3f2c.js.map[39m[22m     619 kB       [1m0[39m[22m  [1m[32m[emitted][39m[22m  vendor
              [1m[32mstatic/js/app.b22ce679862c47a75225.js.map[39m[22m    22.2 kB       [1m1[39m[22m  [1m[32m[emitted][39m[22m  app
         [1m[32mstatic/js/manifest.2ae2e69a05c33dfc65f8.js.map[39m[22m    4.97 kB       [1m2[39m[22m  [1m[32m[emitted][39m[22m  manifest
                                             [1m[32mindex.html[39m[22m  506 bytes        [1m[39m[22m  [1m[32m[emitted][39m[22m  

  Build complete.

  Tip: built files are meant to be served over an HTTP server.
  Opening index.html over file:// won't work.

+ ls -l dist/
total 4
-rw-r--r--    1 root     root           506 Aug 20 01:34 index.html
drwxr-xr-x    4 root     root            27 Aug 20 01:34 static
+ sleep 15
[Pipeline] }
$ docker stop --time=1 ea767dafdc3ba98baf9399ebcde3fd491911d77a0c993f92a8a3b6f44a51fc7a
$ docker rm -f ea767dafdc3ba98baf9399ebcde3fd491911d77a0c993f92a8a3b6f44a51fc7a
```


### 1-5 FAQ

npm构建权限问题：使用root用户构建。设置容器运行用户 `-u 0:0`
