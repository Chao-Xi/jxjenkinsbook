# 第二节 集成 SaltStack 部署工具

## 1、安装saltstack

* https://repo.saltstack.com/index.html#rhel

### 1-1 centos7

```
$ sudo yum install https://repo.saltstack.com/py3/redhat/salt-py3-repo-latest.el7.noarch.rpm

$ sudo yum install salt-master 
$ sudo yum install salt-minion

# 编辑/etc/salt/minion 文件填写对应的master地址
$ sudo vim /etc/salt/minion
master: 192.168.33.11

# 启动master 和 minion
$ sudo service salt-master start
$ sudo chkconfig salt-master on

$ sudo service salt-minion start
$ sudo chkconfig salt-minion on

#在master节点上认证客户端

$ sudo salt-key -L 
Accepted Keys:
Denied Keys:
Unaccepted Keys:
jabox
Rejected Keys:

# salt-key -a clientName
$ sudo salt-key -a jabox
The following keys are going to be accepted:
Unaccepted Keys:
jabox
Proceed? [n/Y] y
Key for minion jabox accepted.

$ sudo salt "jabox" test.ping
jabox:
    True

$ sudo salt jabox cmd.run ls
jabox:
    anaconda-ks.cfg
    original-ks.cfg
```

## 2、集成saltstack

### 2-1 SharedLibrary

**`JenkinslibTest/src/org/devops/deploy.groovy`**

```
package org.devops

//salt stack

def SaltDeploy(host,func){
    sh "sudo salt ${host} ${func}"
}
```

### 2-2 Pipeline

```
#!groovy
@Library('jenkinslib@master') _

def build = new org.devops.buildtools()
def deploy = new org.devops.deploy()

pipeline {
 	agent { node { label "hostmachine" }}
 	parameters {
 		choice(name: 'buildType', choices: 'mvn\nant\ngradle\nnpm', description: 'Please chose your build tool') 
    	choice(name: 'buildShell', choices: '-v\nclean package\nclean install\nclean test', description: 'Please chose your build command') 
    	choice(name: 'deployHosts', choices: 'jabox', description: 'Please chose your salt minion')
	}
 	stages{
		stage('build-deploy') {
	        steps {
	        	script {
	            	build.Build(buildType,buildShell)

	            	deploy.SaltDeploy("${deployHosts}","test.ping")
	            	deploy.SaltDeploy("${deployHosts}","cmd.run ls")
	            } 
	        }
	    }
    }
 }
```

* `def deploy = new org.devops.deploy()`
* `deploy.SaltDeploy("${deployHosts}","test.ping")`
* `deploy.SaltDeploy("${deployHosts}","cmd.run ls")`

### 2-3 Console Output

```
...
[Pipeline] sh
+ sudo salt jabox test.ping
jabox:
    True
[Pipeline] sh
+ sudo salt jabox cmd.run ls
jabox:
    anaconda-ks.cfg
    original-ks.cfg
...
```

**Multiple Hosts Deploy**

```
String deployHosts = "host1,host02" 
stage("Deploy"){
	steps{ 
		script{ 
			hosts = deployHosts.split(",").toList() 
			
			for (host in hosts){ 
				sh "salt ${host} cmd.run ls"
			} 
		} 
	}
}
```
