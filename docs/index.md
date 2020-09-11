# **手摸手 Jenkins 战术教程 (大师版）**

> Started at June 2020 By Jacob Xi 

![Alt Image Text](images/indx1_0.jpg "Body image")

## 内容简介

本书是本人的“手摸手战术教程”系列的第一本，历经三个月的时间，终于在本人30岁之前顺利完工，感谢家人，朋友，同事们的支持与理解，也感谢所在team同事们的帮助与指导。

本书着重介绍了Jenkins 2.x 在实际生产工作中的各种用法与流水线的开发，运维，以及与各个常用生产工具的整合。

本书共15章，主要内容包括：

*  Jenkins的日常运维管理
*  Jenkins流水线的编写与Groovy语法
*  构建工具集成: **Maven, Gradle, Ant, NPM, Ansible, SaltStack**
*  凭证管理: **HashiCorp Vault**
*  用户认证集成: **LADP, Gitlab, Github**
*  代码质量平台集成:  JUnit, JaCoCo, Taurus, **SonarQube**, Allure
*  制品库集成: **Sonatype Nexus, Jfrog Artifactory**
*  需求管理工具集成: Jira, Gitlab
*  容器集成: Docker, Kubernetes
*  自动化接口测试: Jmeter
*  流水线最佳实践: K8S, Gitlab, Maven, Java, Nodejs CI/CD 流水线
*  监控与API调用: Prometheus, InfluxDB, Grafana, Python, REST, CORE API

## Salut! C'est Moi

> The man is not old as long as he is seeking something, A man is not old until regrets take the place of dreams.

Hello, this is me, Jacob. Currently, I'm working as DevOps and Cloud Engineer in SAP, and I'm the certified AWS Solution Architect and Certified Azure Administrator, Kubernetes Specialist and CI/CD enthusiast. 

I was working as Backend Engineer in New York City and achieved my CS master degree in SIT, America. Believe it or not, I'll keep writing, more and more books will come out this year. 

If you have anything want to talk to me directly, you can reach out for via email xichao2015@outlook.com。


Salute, c'est moi, Jacob. Actuellement, je travaille en tant qu'ingénieur DevOps et Cloud dans SAP, et je suis architecte de solution AWS certifié et administrateur Azure certifié, spécialiste Kubernetes et passionné de CI/CD.

Je travaillais en tant qu'ingénieur backend à New York et j'ai obtenu mon master CS à SIT, en Amérique. Croyez-le ou non, je continuerai à écrire, de plus en plus de livres sortiront cette année.

## 目录大纲

* **第一章 Jenkins 运维管理**
	* [第一节 Jenkins 运维管理](https://chao-xi.github.io/jxjenkinsbook/jk_chapter1/)
	* [第二节 分布式构建与并行构建](https://chao-xi.github.io/jxjenkinsbook/jk_chapter1_2/)
	* [第三节 可视化构建试图](https://chao-xi.github.io/jxjenkinsbook/jk_chapter1_3/)
* **第二章 Jenkins 流水线基础**
	* [第一节 编写 Jenkinsfile 运行流水线](https://chao-xi.github.io/jxjenkinsbook/chap2/1chap2_Jenkinsfile/) 
	* [第二节 声明式流水线语法](https://chao-xi.github.io/jxjenkinsbook/chap2/2chap2_declarative_pipeline/)
	* [第三节 脚本化Pipeline](https://chao-xi.github.io/jxjenkinsbook/chap2/3chap2_script_pipeline/)
	* [第四节 Jenkins Pipeline 扩展 —— 共享库](https://chao-xi.github.io/jxjenkinsbook/chap2/4chap2_sharedlib/)
	* [第五节 如何快速上手Jenkinsfile编写](https://chao-xi.github.io/jxjenkinsbook/chap2/5chap2_quick_jenkinsfile/)
	* [第六节 脚本式管道与声明式管道-四个实际差异](https://chao-xi.github.io/jxjenkinsbook/chap2/6chap2_pipe_vs_dec/)
	* [第七节 使用Jenkins Git参数实现分支标签动态选择](https://chao-xi.github.io/jxjenkinsbook/chap2/7chap2_jenkins_git/)
* **第三章 Groovy 基础**
	* [第一节 Groovy 简明教程](https://chao-xi.github.io/jxjenkinsbook/chap3/1chap3_groovy_basic/) 
	* [第二节 Groovy 基础](https://chao-xi.github.io/jxjenkinsbook/chap3/2chap3_groovy_basic2/)
* **第四章 构建工具集成**
	* [第一节 构建工具集成](https://chao-xi.github.io/jxjenkinsbook/chap4/1chp4_tools1/) 
	* [第二节 集成 SaltStack 部署工具](https://chao-xi.github.io/jxjenkinsbook/chap4/2chp4_tools2/)
	* [第三节 Jenkins集成Ansibe实现自动化部署](https://chao-xi.github.io/jxjenkinsbook/chap4/3chp4_tools3/)
	* [第四节 Jenkins集成Ansibe实战手册 —— BB Mobile（2018）](https://chao-xi.github.io/jxjenkinsbook/chap4/4chp4_tools4/)
* **第五章 凭证管理**
	* [第一节 凭证管理](https://chao-xi.github.io/jxjenkinsbook/chap5/1chap5_cred1/) 
	* [第二节 凭证管理使用HashiCorp Vault](https://chao-xi.github.io/jxjenkinsbook/chap5/2chap5_vault/)
	* [第二节(Extra) - Store Secrets using Hashicorp Vault with Docker](https://chao-xi.github.io/jxjenkinsbook/chap5/3docker_valut/)
	* [第三节 在Jenkins日志中隐藏敏感信息](https://chao-xi.github.io/jxjenkinsbook/chap5/4docker_mask/)
* **第六章 用户认证集成**
	* [第一节 Ldap 用户认证集成](https://chao-xi.github.io/jxjenkinsbook/chap6/1chap6_ldap/)
	* [第二节 Jenkins集成Gitlab SSO单点登录](https://chao-xi.github.io/jxjenkinsbook/chap6/2chap6_gitlab/)
	* [第三节 Jenkins集成Github SSO单点登录](https://chao-xi.github.io/jxjenkinsbook/chap6/3chap6_github/)
* **第七章 Gitlab 版本控制系统集成**
	* [第一节 Gitlab – Jenkins Integration](https://chao-xi.github.io/jxjenkinsbook/chap7/1chap7_gitlab_connect/)
	* [第二节 Gitlab-Jenkins 版本控制](https://chao-xi.github.io/jxjenkinsbook/chap7/2chap7_gitlab_pipeline/)
	* [第三节 第三节 Gitlab-Jenkins 版本控制（补充）](https://chao-xi.github.io/jxjenkinsbook/chap7/3chap7_gitlab_pipeline_extra/)
* **第八章 代码质量平台集成**
	* [第一节 代码质量](https://chao-xi.github.io/jxjenkinsbook/chap8/1chap8_code_quality/)
	* [第二节 SonarQube：持续代码质量检查](https://chao-xi.github.io/jxjenkinsbook/chap8/2chap8_code_quality_sonarqube/)
	* [第三节 Allure测试报告：更美观的测试报告](https://chao-xi.github.io/jxjenkinsbook/chap8/3chap8_code_quality_allure/)
	* [第四季 代码质量平台集成(SonarQube Final Edition)](https://chao-xi.github.io/jxjenkinsbook/chap8/4chap8_code_quality_integration/)
* **第九章 制品库集成**
	* [第一节 Sonatype Nexus 基本介绍与安装](https://chao-xi.github.io/jxjenkinsbook/chap9/chap9_artifact0/) 
	* [第二节 制品管理](https://chao-xi.github.io/jxjenkinsbook/chap9/chap9_artifact1/)
	* [第三节 Nexus 制品上传](https://chao-xi.github.io/jxjenkinsbook/chap9/chap9_artifact2/)
	* [第四节 Jenkins & Jfrog Artifactory](https://chao-xi.github.io/jxjenkinsbook/chap9/chap9_artifact3/)
* **第十章 需求管理工具集成**
	* [第一节 k8s 安装本地 Jira 服务](https://chao-xi.github.io/jxjenkinsbook/chap10/1chap10_jira_install/) 
	* [第二节 Jenkins 配置端到端流水线 （Jira 和 Gitlab的集成）](https://chao-xi.github.io/jxjenkinsbook/chap10/2chap10_jira_gilab/)
* **第十一章 Docker 集成**
	* [第一节 基于 Docker 配置构建资源池](https://chao-xi.github.io/jxjenkinsbook/chap11/1Docker_agents/) 
	* [第二节 基于Docker的pipeline流水线](https://chao-xi.github.io/jxjenkinsbook/chap11/2Docker_pipeline/)
	* [第三节 构建应用镜像到镜像仓库管理](https://chao-xi.github.io/jxjenkinsbook/chap11/3Docker_registry/)
	* [第四节 使用Groovy代码初始化Docker配置](https://chao-xi.github.io/jxjenkinsbook/chap11/4Docker_groovy/)
* **第十二章 K8S 平台集成**
	* [第一节 K8S 平台集成(全)](https://chao-xi.github.io/jxjenkinsbook/chap12/1k8s_install/)
* **第十三章 自动化接口测试**
	* [第一节 Jmeter & Ant 自动化测试](https://chao-xi.github.io/jxjenkinsbook/chap13/1jmeter/)
	* [第二节 Jenkins、Ant、Jmeter 自动化测试](https://chao-xi.github.io/jxjenkinsbook/chap13/2jenkins_jmeter/)
* **第十四章 流水线最佳实践**
	* [第一节 基于Jira端到端流水线的最佳实践](https://chao-xi.github.io/jxjenkinsbook/chap14/1JIRA_JEN_K8S/)
	* [第二节 Jenkins K8S Gitlab 集成](https://chao-xi.github.io/jxjenkinsbook/chap14/2JEN_K8S_API/)
	* [第三节 Jenkins + K8S + Gitlab 构建 RLEASE 打包发布更新流水线到K8S集群](https://chao-xi.github.io/jxjenkinsbook/chap14/3Jen_k8s_pipeline/)
	* [第四节 版本晋级及发布流水线](https://chao-xi.github.io/jxjenkinsbook/chap14/4Jen_update_deploy_pip/)
	* [第五节 前端后端项目发布流水线（Java+Nodejs)](https://chao-xi.github.io/jxjenkinsbook/chap14/5back_front_pipeline/)
* **第十五章 监控与API调用**
	* [第一节 使用 Prometheus 监控 Jenkins](https://chao-xi.github.io/jxjenkinsbook/chap15/1Jenkins_Prometheus/)
	* [第二节 Jenkins+InfluxDB+Grafana 收集构建数据](https://chao-xi.github.io/jxjenkinsbook/chap15/2Jenkins_InfluxDB_Grafana/)
	* [第三节 Jenkins Python API 实践](https://chao-xi.github.io/jxjenkinsbook/chap15/3Jenkins_python_API/)
	* [第四节 Jenkins REST API 使用实践](https://chao-xi.github.io/jxjenkinsbook/chap15/4Jenkins_REST_API/)
	* [第五节 Jenkins Core Api & Job DSL创建项目](https://chao-xi.github.io/jxjenkinsbook/chap15/5Jenkins_DSL/)

### Reference Code:

* [JenkinslibTest(Shared Library functions)](https://github.com/Chao-Xi/JenkinslibTest)
* [Jenkinsfiles for the book](https://github.com/Chao-Xi/JenkinslibTest/tree/master/Jenkinsfiles)
* [jenkins-maven-service(Gitlab: demo-maven-service)](https://github.com/Chao-Xi/jenkins-maven-service)
* [jmetertest(Gitlab: demo-interfacetest)](https://github.com/Chao-Xi/jmetertest)


## To be continue

本人将带来手摸手战术教程更多的内容和文章， 接下来的将在ES7, Chef, Azure900, Azure103, AWS Solution Arcitect, AWS Big Data Speciality, Istio, Python带来更多更全面的电子书，敬请期待。(PS: 下面的图片单纯为了好玩)

![Alt Image Text](images/indx1_1.png "Body image")