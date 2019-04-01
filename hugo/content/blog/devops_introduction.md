+++
date = 2019-03-29T14:43:22+08:00
title = "DevOps 简介"
description = "DevOps 简介"
slug = "" 
tags = ["docker", "DevOps", "kubernets"]
categories = []
externalLink = ""
series = ["docker", "DevOps", "kubernets"]
+++




# 目标
1. 打造通用, 稳定, 快速, 易于扩展的部署流水线
    - 部署流水线是指软件从版本控制库到用户手中这一过程的自动化表现形式。对软件的每次变更都会经历这样一个自动化的过程
    - 部署流水线包含代码检查, 持续集成, 持续部署, 自动化测试等各个环节
2. 构建自助平台
    - 各项目组可一键创建测试所以需要的容器化Java环境, 数据库等. 并可通过自助平台进行管理
    - 通过自助平台配置自动构建行为
    - 集成日志查看功能

# 参考示例
零件俱乐部项目的部署流程如下图所示, 开发人员无需关心部署的细节, 只需要推送代码到对应的分支, 就会触发部署流水线部署到相应的环境.

![](/images/devops_introduction/publish-workflow.png)

1. 开发人员使用[特性分支](https://git.gantcloud.com/public-group/git-tutorial/wikis/git-feature-based-flow)的git分支模型来开发新功能, 代码需要推送到[GitLab](https://docs.gitlab.com/ee/)仓库
2. GitLab根据代码仓库中的gitlab-ci.yaml文件来, 触发[GitLab CI](https://docs.gitlab.com/ee/ci/)执行 `编译->测试->发布镜像->部署` 流水线
    - 特定的处理环节(比如多个feature分支的自动合并, 版本号的确定) 采用[Shell](https://www.tutorialspoint.com/unix/shell_scripting.htm)或者[Python](https://www.python.org/)脚本来实现
    - Java代码的构建会用到[Gradle](https://gradle.org/), js代码的构建会用到[npm](https://www.npmjs.com/)或[yarn](https://yarn.bootcss.com/)
    - 执行单元测试, 如果测试失败会终止构建, 并使用邮件通知给本次构建的触发者
    - 编译完成之后从代码仓库中获取[Dockerfile](https://docs.docker.com/engine/reference/builder/)文件, 然后使用[Docker](https://docs.docker.com/v17.09/engine/docker-overview/)制作成镜像文件
3. 发布到[Harbor](https://goharbor.io/), harbor是docker镜像的仓库, 基于官方的[Registry](https://docs.docker.com/registry/)
4. 通过kubectl触发[Kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)执行发布流程
5. k8s拉取指定镜像, 进行[滚动升级](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)

# 书单

## DevOps方法论

- [《凤凰项目：一个IT运维的传奇故事》](https://book.douban.com/subject/26644070/)
以小说的方式，讲述了一个凌乱的无可救药的运维项目组是如何一步步达成最后高效且舒心的工作状态。

- [《DevOps实践指南》](https://book.douban.com/subject/30186150/)
《凤凰项目：一个IT运维的传奇故事》 的姊妹篇， 涵盖了DevOps的横向知识，可以当工具书读。

- [《持续交付》](https://book.douban.com/subject/6862062/)

- [《Effective DevOps》](https://book.douban.com/subject/26631259/)

## 源代码管理/Linux系统

- [《git pro》](https://git-scm.com/book/zh/v2) 
网络资源， 免费

- [《Unix & Linux大学教程》](https://book.douban.com/subject/4253716/)
Linux, Shell 学习的经典书籍

## 容器化：
- [《第一本Docker书》](https://book.douban.com/subject/26285268/)
- [《自己动手写Docker》](https://book.douban.com/subject/27082348/)

## 容器编排
- [《Kubernetes中文指南》](https://jimmysong.io/kubernetes-handbook/cloud-native/kubernetes-and-cloud-native-app-overview.html)
- [《Kubernetes in Action》](https://book.douban.com/subject/26997846/)
- [官网教程](https://kubernetes.io/docs/tutorials/)

## 分布式存储
- [《Learning Ceph》](https://book.douban.com/subject/26336220/)
- [《Ceph设计原理与实现》](https://book.douban.com/subject/27178824/)

## 自动化测试

(待补充) 单元测试, ui自动化测试 等

## 自动化运维
Ansible

## 日志
Elasticsearch, Logstash/Fluentd, Kibana

## 监控
Prometheus

# 路线图
![](/images/devops_introduction/15134301.png)

