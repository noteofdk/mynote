---
title: "Docker compose note (ing)"
date: 2016-07-25 23:35:25
categories: [docker]
tags: [doker, compose]
---

Docker Compose 是定义、构建多容器的 docker 应用的工具。
Compose 通过 compose 文件配置应用的服务。之后，用一条指令就可以从配置创建、启动所有服务。 

<!--more-->

和 CI 工作流程搭配，Compose 在开发、测试、模拟环境下非常给力。更多内容参见 [Common Use Cases.](https://docs.docker.com/compose/overview/#common-use-cases)。

使用 Compose 基于一下三步：

- 1. 用 Dockerfile 定义应用的环境（以在任何环境下都能重现）
- 2. 在 docker-compose.yml 中定义组成应用的服务（services）（以让所有服务能在孤立的环境一起运行）
- 3. 最后，运行 `docker-compose up` ，Compose 会启动，接着运行整个应用。

[原文地址](https://docs.docker.com/compose/overview/)

docker-compose.yml 格式如下：

``` yml
version: '2'
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
Compose 文件详情参见：[Compose file reference](https://docs.docker.com/compose/compose-file/)

Compose 有管理应用的整个生命周期的指令：

- Start, stop and rebuild services
- View the status of running services
- Stream the log output of running services
- Run a one-off command on a service


## [特性（Features）](https://docs.docker.com/compose/overview/#/features)

- Multiple isolated environments on a single host
- Preserve volume data when containers are created
- Only recreate containers that have changed
- Variables and moving a composition between environments

## Common Use Cases
几种常见用例如下：

### 开发环境
开发软件时，在隔离的环境下运行应用和进行交互的能力至关重要。Compose 指令工具可用来创建这种环境并与其交互。

Compose file 提供归档（document）并配置应用的所有服务的依赖（数据库，队列，缓存，web服务API 等等）。使用 Compose 指令工具可以用单独的一行指令（`docker-compose up`）为每个依赖创建并启动一个或多个容器。

总之，这些特性为开发者提供一条开始开发项目的便捷途径。

### 自动化测试环境
任何持续部署或者集成过程的重要一部分是自动测试组件。自动化的端到端测试要求一个运行测试的环境。Compose 为测试组件提供一个方便地创建、销毁隔离地测试环境的途径。通过在 Compose file 中定义整个环境，可用几行指令就创建、销毁这些环境。

``` shell
$ docker-compose up -d
$ ./run_tests
$ docker-compose down
```

### 单一主机的部署
传统地，将 Compose 定位为部署、测试工作流，但每个发布版都展现出更多的生产导向性。可以用 Compose 部署到一个远程 Docker 引擎。该 Docker 引擎可能是一个由 Docker Machine 预分配的单独实例，也可能是整个 Docker Swarm 集群。

更多生产导向性特性，见 [compose in production](https://docs.docker.com/compose/production/)


# [安装 Compose]((https://docs.docker.com/compose/install/))

1. 安装 Docker
2. github 上 Compose repository 发行版页面
3. 安装命令：
```curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose```
4. 为二进制文件添加执行权限：
``` $ chmod +x /usr/local/bin/docker-compose ```
5. 检测安装
```
$ docker-compose --version
docker-compose version: 1.7.1 
```

## Alternative install options
### Install using pip
Compose can be installed from pypi using pip. If you install using pip it is highly recommended that you use a virtualenv because many operating systems have python system packages that conflict with docker-compose dependencies. See the virtualenv tutorial to get started.

```
$ pip install docker-compose
```
> Note: pip version 6.0 or greater is required

### Install as a container
Compose can also be run inside a container, from a small bash script wrapper. To install compose as a container run:

```
$ curl -L https://github.com/docker/compose/releases/download/1.7.1/run.sh > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

## Master builds
If you’re interested in trying out a pre-release build you can download a binary from https://dl.bintray.com/docker-compose/master/. Pre-release builds allow you to try out new features before they are released, but may be less stable.

## Upgrading
If you’re upgrading from Compose 1.2 or earlier, you’ll need to remove or migrate your existing containers after upgrading Compose. This is because, as of version 1.3, Compose uses Docker labels to keep track of containers, and so they need to be recreated with labels added.

If Compose detects containers that were created without labels, it will refuse to run so that you don’t end up with two sets of them. If you want to keep using your existing containers (for example, because they have data volumes you want to preserve) you can use compose 1.5.x to migrate them with the following command:

    $ docker-compose migrate-to-labels
Alternatively, if you’re not worried about keeping them, you can remove them. Compose will just create new ones.

    $ docker rm -f -v myapp_web_1 myapp_db_1 ...
## Uninstallation
To uninstall Docker Compose if you installed using curl:

    $ rm /usr/local/bin/docker-compose
To uninstall Docker Compose if you installed using pip:

    $ pip uninstall docker-compose
> Note: If you get a “Permission denied” error using either of the above methods, you probably do not have the proper permissions to remove docker-compose. To force the removal, prepend sudo to either of the above commands and run again.


# [Getting Started](https://docs.docker.com/compose/gettingstarted/)

# [Get started with Django](https://docs.docker.com/compose/django/)

# [Get started with Rails](https://docs.docker.com/compose/rails/)

# [Get started with WordPress](https://docs.docker.com/compose/wordpress/)

# [Frequently asked questions](https://docs.docker.com/compose/faq/)

# [Command line reference](https://docs.docker.com/compose/reference/)

# [Compose file reference](https://docs.docker.com/compose/compose-file/)

# [Environment file](https://docs.docker.com/compose/env-file/)
