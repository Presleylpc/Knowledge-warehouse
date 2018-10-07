# DockerCompose 概述

[官方原文地址](https://docs.docker.com/compose/overview/)

> `DockerCompose`最主要的作用，是在一台服务器上进行集群管理。

Compose是一个定义、运行多个Docker容器的工具。使用Compose时候，你将使用一个YAML格式的文件来配置你的应用服务。然后，一条命令就可以将你定义的所有应用服务启动起来。

使用Compose有三个基本步骤

1. 在`Dockerfile`中定义你的app的运行环境，这样你就可以在任何地方启用。

2. 在`docker-compose.yml`中定义构成应用程序的服务，以便它们可以在一个隔离环境中一起运行。

3. 执行`docker-compose up`，Compose将启动并运行整个应用程序。

一个`docker-compose.yml`格式如下:

```yaml
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

关于 Compose文件的更多信息，可以查看[ 04-3 Compose file reference]()

Compose 还可以管理应用的整个生命周期

- 启动，停止和重建服务
- 查看正在运行的服务的状态
- 流式传输运行服务的日志输出
- 在服务上运行一次性命令

## Compose 特性

Compose的特性如下:

- 单个主机上的多个隔离环境
- 创建容器时保留卷数据
- 仅重新创建配置修改过的容器
- Variables and moving a composition between environments

### 单个主机上的多个隔离环境

Compose使用项目名称将环境彼此隔离。您可以在几个不同的上下文中使用此项目名称：

- 在开发主机上，创建单个环境的多个副本，例如当您要为项目的每个功能分支运行稳定副本时
- 在CI服务器上，为了防止构建相互干扰，可以将项目名称设置为唯一的构建号
- 在共享主机或开发主机上，以防止可能使用相同服务名称的不同项目相互干扰

默认项目名称是项目目录的基名。您可以使用[`-p`命令行选项](https://docs.docker.com/compose/reference/overview/)或[`COMPOSE_PROJECT_NAME`环境变量](https://docs.docker.com/compose/reference/envvars/#compose-project-name)设置自定义项目名称。

### 创建容器时保留卷数据

Compose会保留您的服务使用的所有卷。当`docker-compose up`运行时，如果发现任何集装箱从之前的运行，它会将从旧容器到新容器的体积。此过程可确保您在卷中创建的任何数据都不会丢失。

如果`docker-compose`在Windows计算机上使用，请参阅[环境变量](https://docs.docker.com/compose/reference/envvars/)并根据您的特定需求调整必要的环境变量。

### 仅重新创建已更改的容器

Compose缓存用于创建容器的配置。当您重新启动未更改的服务时，Compose将重新使用现有容器。重用容器意味着您可以非常快速地更改环境。

### **Variables and moving a composition between environments**

Compose支持Compose文件中的变量。您可以使用这些变量为不同环境或不同用户自定义组合。有关详细信息，请参阅[变量替换](https://docs.docker.com/compose/compose-file/#variable-substitution)

您可以使用该`extends`字段或通过创建多个Compose文件来扩展Compose文件。有关详细信息，请参阅[extends](https://docs.docker.com/compose/extends/)。

## 常见用例

Compose可以以多种不同的方式使用。下面概述了一些常见用例。

### 开发环境

在开发软件时，在隔离环境中运行应用程序并与之交互的能力至关重要。Compose命令行工具可用于创建环境并与之交互。

在[撰写文件](https://docs.docker.com/compose/compose-file/)提供了一种记录和配置所有应用程序的服务依赖（数据库，队列，高速缓存，Web服务的API，等等）。使用Compose命令行工具，您可以使用单个命令（`docker-compose up`）为每个依赖项创建和启动一个或多个容器。

这些功能共同为开发人员提供了一个开始项目的便捷方式。Compose可以将多页“开发人员入门指南”减少到单个机器可读的Compose文件和一些命令。

### 自动化测试环境

任何持续部署或持续集成过程的一个重要部分是自动化测试套件。自动化端到端测试需要一个运行测试的环境。Compose提供了一种方便的方法来为您的测试套件创建和销毁隔离的测试环境。通过在[Compose文件中](https://docs.docker.com/compose/compose-file/)定义完整环境，您可以在几个命令中创建和销毁这些环境：

```
$ docker-compose up -d
$ ./run_tests
$ docker-compose down
```

### 单主机部署

Compose传统上一直专注于开发和测试工作流程，但每个版本我们都在更多面向生产的功能上取得进展。您可以使用Compose部署到远程Docker引擎。Docker Engine可以是使用[Docker Machine](https://docs.docker.com/machine/overview/)或整个[Docker Swarm](https://docs.docker.com/engine/swarm/)集群配置的单个实例。

有关使用面向生产的功能的详细信息，请参阅本文档[中的生产](https://docs.docker.com/compose/production/)中的[撰写](https://docs.docker.com/compose/production/)。

## 发行说明

要查看Docker Compose的过去和当前版本的更改的详细列表，请参阅[CHANGELOG](https://github.com/docker/compose/blob/master/CHANGELOG.md)。

## 获得帮助

Docker Compose正在积极开发中。如果您需要帮助，想要贡献，或者只是想与志同道合的人谈论项目，我们有一些开放的沟通渠道。

- 要报告错误或文件功能请求：[在Github上](https://github.com/docker/compose/issues)使用[问题跟踪器](https://github.com/docker/compose/issues)。

- 与人们实时讨论该项目：加入`#docker-compose`freenode IRC的频道。

- 要贡献代码或文档更改：[在Github上](https://github.com/docker/compose/pulls)提交[拉取请求](https://github.com/docker/compose/pulls)。

有关更多信息和资源，请访问“[获得帮助”项目页面](https://docs.docker.com/opensource/get-help/)。


