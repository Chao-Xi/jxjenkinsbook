# **第二节 Jenkins K8S Gitlab 集成**

文档： https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#deployment-v1-apps


## 1、Curl K8S-API Without kubectl proxy 

https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/

```
$ kubectl config view -o jsonpath='{"Cluster name\tServer\n"}{range .clusters[*]}{.name}{"\t"}{.cluster.server}{"\n"}{end}'
Cluster name    Server
docker-desktop  https://kubernetes.docker.internal:6443

# Point to the API server referring the cluster name
kubectl config view -o jsonpath="{.clusters[?(@.name==\"docker-desktop\")].cluster.server}"
https://kubernetes.docker.internal:6443

# Gets the token value
kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 --decode


curl -X GET https://kubernetes.docker.internal:6443/api --header "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkJLazJRem5PZVpBOHoyVjRHUFZtX2w4VURDcjNEQ2xucVdoRGF5QldxR00ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tYjVxa3YiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImVjNDMwNjYxLTkxYzktNDZlYS05YzllLWJmYWFkNTA2MmIwZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.vXOTW2DaIZhOzcIazOzCuI2Kc_BBQcfJ77sZpT_vnAYsPledlhb_-pUYZblxTtzomQYHRDSKw_Ffsp-wHpFcX2R_g2N8ww0MZ-Np3nv7T-_vAdExhyRVyYcfwDcA3l4QFxjTLFM9qUAMhqlnrBAbkfE1gNNA6IzWwwrftu4L42BPBLa4nzb_dxjYMHUoKhlj79AoLvPDaeLVicl6YXunEZlzqYDNe3z9fbiO_c2h7dYf70FWO34q5GGqvc0yItPlpl2w4irxviK_0j_kVEn-7uIHwGW66CH5i-vJazh7jQmJG9dapBVN9lg00MrD-q3zcV1O2WWpmvUacR6bauWe9A" --insecure
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "192.168.65.3:6443"
    }
  ]
}

curl --header "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IkJLazJRem5PZVpBOHoyVjRHUFZtX2w4VURDcjNEQ2xucVdoRGF5QldxR00ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tYjVxa3YiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImVjNDMwNjYxLTkxYzktNDZlYS05YzllLWJmYWFkNTA2MmIwZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.vXOTW2DaIZhOzcIazOzCuI2Kc_BBQcfJ77sZpT_vnAYsPledlhb_-pUYZblxTtzomQYHRDSKw_Ffsp-wHpFcX2R_g2N8ww0MZ-Np3nv7T-_vAdExhyRVyYcfwDcA3l4QFxjTLFM9qUAMhqlnrBAbkfE1gNNA6IzWwwrftu4L42BPBLa4nzb_dxjYMHUoKhlj79AoLvPDaeLVicl6YXunEZlzqYDNe3z9fbiO_c2h7dYf70FWO34q5GGqvc0yItPlpl2w4irxviK_0j_kVEn-7uIHwGW66CH5i-vJazh7jQmJG9dapBVN9lg00MrD-q3zcV1O2WWpmvUacR6bauWe9A" --insecure  -X GET https://kubernetes.docker.internal:6443/api/v1/nodes

{
  "kind": "NodeList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/nodes",
    "resourceVersion": "1044977"
  },
  "items": [
    {
      "metadata": {
        "name": "docker-desktop",
        "selfLink": "/api/v1/nodes/docker-desktop",
        "uid": "92d70f98-883a-4cd3-8b54-35a6187eab88",
        "resourceVersion": "1044565",
        ...
}

```

## 2、Jenkins Pipeline 调用 K8S_API

### 2-1 `kubernetets.groovy` 

```
package org.devops


//封装HTTP请求
def HttpReq(reqType,reqUrl,reqBody){
    def apiServer = "https://kubernetes.docker.internal:6443/apis/apps/v1"
    withCredentials([string(credentialsId: 'kubernetes-token', variable: 'kubernetestoken')]) {
      result = httpRequest customHeaders: [[maskValue: true, name: 'Authorization', value: "Bearer ${kubernetestoken}"],
                                           [maskValue: false, name: 'Content-Type', value: 'application/yaml'], 
                                           [maskValue: false, name: 'Accept', value: 'application/yaml']], 
                httpMode: reqType, 
                consoleLogResponseBody: true,
                ignoreSslErrors: true, 
                requestBody: reqBody,
                url: "${apiServer}/${reqUrl}"
                //quiet: true
    }
    return result
}
//新建Deployment
def CreateDeployment(nameSpace,deployName,deplyBody){
    apiUrl = "namespaces/${nameSpace}/deployments/"
    response = HttpReq('POST',apiUrl,deplyBody)
    println(response)
}

//删除deployment
def DeleteDeployment(nameSpace,deployName){
    apiUrl = "namespaces/${nameSpace}/deployments/${deployName}"
    response = HttpReq('DELETE',apiUrl,deplyBody)
    println(response)
}

//更新Deployment
def UpdateDeployment(nameSpace,deployName,deplyBody){
    apiUrl = "namespaces/${nameSpace}/deployments/${deployName}"
    response = HttpReq('PUT',apiUrl,deplyBody)
    println(response)
}

//获取Deployment
def GetDeployment(nameSpace,deployName){
    apiUrl = "namespaces/${nameSpace}/deployments/${deployName}"
    response = HttpReq('GET',apiUrl,'')
    return response
}
```

**可配信息**

* `kubernetes.docker.internal`: Cluster Server
* `kubernetes-token`

### 2-2 K8S Jenkins 配置 k8s 信息

我们在**集群内部的Jenkins**配置`k8s-token`，`SharedLibrary` 等等信息

**SharedLibrary**

* jenkinslib
* master
* https://github.com/Chao-Xi/JenkinslibTest.git
* Credentials	: github

![Alt Image Text](../images/chp14_2_1.png "body image")


**kubernetes-token**

![Alt Image Text](../images/chp14_2_2.png "body image")


### 2-3 创建测试`k8s` 服务

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: demo-uat
---
apiVersion: v1
kind: Namespace
metadata:
  name: demo-stage
---
apiVersion: v1
kind: Namespace
metadata:
  name: demo-prod
```

```
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: demoapp
  name: demoapp
  namespace: demo-prod
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: demoapp
  template:
    metadata:
      labels:
        k8s-app: demoapp
      namespace: demo-prod
      name: demoapp
    spec:
      containers:
        - name: demoapp
          image: bitnami/java-example:0.0.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
              name: web
              protocol: TCP
      serviceAccountName: demoapp
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: demoapp
  name: demoapp
  namespace: demo-prod
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: demoapp
  name: demoapp
  namespace: demo-prod
spec:
  type: NodePort
  ports:
    - name: web
      port: 8080
      targetPort: 8080
      nodePort: 30991
  selector:
    k8s-app: demoapp
```

```
http://127.0.0.1:30991/
```


### 2-4  K8S Jenkins 调用 k8s-API Pipeline

```
#!groovy
@Library('jenkinslib@master') _

def k8s = new org.devops.kubernetes()

pipeline {
    agent { node { label "master" }}
    stages{
        stage("GetDeployment"){
            steps{
                script{
                    k8s.GetDeployment("demo-prod","demoapp")
                }  
            }
        }
    }
}
```

**Console output**

```
HttpMethod: GET
URL: https://kubernetes.docker.internal:6443/apis/apps/v1/namespaces/demo-prod/deployments/demoapp
Authorization: *****
Content-Type: application/yaml
Accept: application/yaml
Sending request to url: https://kubernetes.docker.internal:6443/apis/apps/v1/namespaces/demo-prod/deployments/demoapp
Response Code: HTTP/1.1 200 OK
Response: 
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"k8s-app":"demoapp"},"name":"demoapp","namespace":"demo-prod"},"spec":{"replicas":1,"revisionHistoryLimit":10,"selector":{"matchLabels":{"k8s-app":"demoapp"}},"template":{"metadata":{"labels":{"k8s-app":"demoapp"},"name":"demoapp","namespace":"demo-prod"},"spec":{"containers":[{"image":"bitnami/java-example:0.0.1","imagePullPolicy":"IfNotPresent","name":"demoapp","ports":[{"containerPort":8080,"name":"web","protocol":"TCP"}]}],"serviceAccountName":"demoapp"}}}}
  creationTimestamp: "2020-08-27T06:13:32Z"
...
  observedGeneration: 3
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1

Success code from [100‥399]
...
```

## 3、 上传 K8S 部署文件到 Gitlab

### 3-1 在新的`Jenkins` 添加 `gitlab-token`


**注意token的类型一定是`secret text`** 否则会报错

```
ERROR: Credentials 'gitlab-token' is of type 'GitLab API token' where 'org.jenkinsci.plugins.plaincredentials.StringCredentials' was expected
```

* **sdsxxZ7ymzoTKSGZ6dNv**


![Alt Image Text](../images/chp14_2_3.png "body image")


### 3-2 `gitlab` 中添加创建仓库文件方法

```
//创建仓库文件
def CreateRepoFile(projectId,filePath,fileContent){
    apiUrl = "projects/${projectId}/repository/files/${filePath}"
    reqBody = """{"branch": "master","encoding":"base64", "content": "${fileContent}", "commit_message": "create a new file"}"""
    response = HttpReq('POST',apiUrl,reqBody)
    println(response)
}
```

新建文件: `projects/${projectsId}/repository/files/${filepath}`

* `testText.bytes.encodeBase64().toString()`
* `decoded = base64text.decodeBase64()`


### 3-3 Pipeline 中调用 CreateRepoFile  上传 k8s.yaml 到 gitlab

```
#!groovy
@Library('jenkinslib@master') _

def k8s = new org.devops.kubernetes()
def gitlab = new org.devops.gitlab()

pipeline {
    agent { node { label "master" }}
    parameters {
        choice(name: 'versionName', choices: '1.0\n1.1\n1.2\n1.3', description: 'Please chose your versionName')
	}
    stages{
        stage("GetDeployment"){
            steps{
                script{
                    response = k8s.GetDeployment("demo-prod","demoapp")
                    response = response.content

                    //文件转换
                    base64Content = response.bytes.encodeBase64().toString()

                    //上传文件
                    gitlab.CreateRepoFile(3,"demo-prod%2f${versionName}-prod.yaml",base64Content)

                }  
            }
        }
    }
}
```

* projectId: 3
* filePath： `"demo-prod%2f${versionName}-prod.yaml"`
* `base64Content`


![Alt Image Text](../images/chp14_2_4.png "body image")

**Console output**

```
...
[Pipeline] // withCredentials
[Pipeline] withCredentials
Masking supported pattern matches of $gitlabToken
[Pipeline] {
[Pipeline] httpRequest
HttpMethod: POST
URL: http://192.168.33.1:30088/api/v4/projects/3/repository/files/demo-prod%2f1.0-prod.yaml
Content-Type: application/json
PRIVATE-TOKEN: *****
Sending request to url: http://192.168.33.1:30088/api/v4/projects/3/repository/files/demo-prod%2f1.0-prod.yaml
Response Code: HTTP/1.1 201 Created
Response: 
{"file_path":"demo-prod/1.0-prod.yaml","branch":"master"}
Success code from [100‥399]
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] echo
Status: 201
```

![Alt Image Text](../images/chp14_2_5.png "body image")



