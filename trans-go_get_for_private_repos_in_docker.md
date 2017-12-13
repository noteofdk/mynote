---
title: "【翻译】docker 中拉取私库"
date: 2016-10-27 9:35:25
categories: 翻译
tags: [golang, docker, ssh]

---

Ivan Daniluk 是一位我很崇敬的大牛。该文翻译自他的一篇博文。该文研究了 docker 中拉取 github 私有仓库时涉及的权限处理问题。涵盖 go 包管理、docker、ssh 等。
<!--more-->

[博客](https://divan.github.io/)

[原文: divan's blog](https://divan.github.io/posts/go_get_private/)

> 插入视频：[GopherCon 2016: Ivan Danyliuk - Visualizing Concurrency in Go](https://youtu.be/KyuFeiG3Y60) 

虽然 go 社区在依赖管理方面一直朝着稳定、易于理解的模式和实践发展，但是依然有很多令人疑惑的地方。其中之一就发生在用容器进行涉及私有仓库依赖的自动可重复的构建的过程中。

当用 go get 时，Github 私库经常是疑惑的源头。但有个很容易的变通办法，只需要在 `.gitconfig` 中增加两行：

```
[url "git@github.com:"]
	insteadOf = https://github.com/
```

或者一行解决：
```
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

当在容器中进行构建时，发生了最令人疑惑的事情。下面以当前最流行的容器 Docker 为例说明。

# 问题
假设有两个包 `github.com/company/foo` 和 `github.com/company/bar`，`foo` 中导入了 `bar`。

```
foo.go:
import "github.com/company/bar"
```

正常的流程是：设置 GOPATH、SSH 密钥对、gitconfig 就好了，简单的命令 `go get github.com/company/foo` 能搞定一切，下载两个包。

```
$ go get -v github.com/company/foo
github.com/company/foo (download)
github.com/company/bar (download)
```

但是现在，我们想让这个构建过程在任何设备上可复现，甚至在一个 CI 实例上，故而选择在 Docker 容器中打包一切。可以用基于官方 golang 的 Dockerfile：

**Dockerfile**
```
FROM golang:1.6

ADD . /go/src/github.com/company/foo
CMD cd /go/src/github.com/company/foo; go get github.com/company/bar && go build -o /go/bin/foo
```

**Build script**
```
docker build -t foo-build . 				# build image
docker run --name=foo-build foo-build		# compile binary
docker cp foo-build:/foo foo				# copy binary to fs
docker rm -f foo-build						# remove container
docker rmi -f foo-build						# remove image
```

这种方式不奏效，因为用于构建（foo-build）的 Docker 容器没有包含 `bar` 的依赖、`ssh` 密钥对以及合适的 gitconfig。而且很明显，简单地增加密钥对并不可行（it’s not trivial simply to add the keys），因为你必须处理以 SSH 为主的一大堆障碍。那，我们就尽快搞定吧。

# 解决办法
## ssh 与 https
首先，构建（build）阶段（`docker run ...`）会遇到如下错误：

```
# cd .; git clone https://github.com/company/bar /go/src/github.com/company/bar
Cloning into '/go/src/github.com/company/bar'...
fatal: could not read Username for 'https://github.com': No such device or address
package github.com/company/bar: exit status 128
```

意思是：你 Github 的权限允许用 SSH 密钥，但 `go get` 援引的 `git` 命令试图通过 HTTPS 途径克隆仓库，而你却没有配置证书。

变通办法很简单，该问开始就有描述。我们只需要在 Dockerfile 中的 `go get` 前增加：

```
RUN echo "[url \"git@github.com:\"]\n\tinsteadOf = https://github.com/" >> /root/.gitconfig
```

## 密钥
下一个错误是主机 key 认证错误：
```
# cd .; git clone https://github.com/company/bar /go/src/github.com/company/bar
Cloning into '/go/src/github.com/company/bar'...
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
package github.com/company/bar: exit status 128
```

这是因为我们的 Docker 容器还没有 SSH 密钥。the right approach is not trivial, 我们继续。

首先，我们想要每个开发者或者 CI 使用自己的私库的密钥。如果一个人有 `foo` 的权限，那么他肯定有 `bar` 的权限，密钥通常在 `~/.ssh/id_rsa` 文件。

然而，不可以将密钥复制到容器。 Dockerfile 的 ADD 和 COPY 命令只能复制当前目录的文件，故而无法直接用 `ADD ~/.ssh/ /root/ssh`。解决办法之一是写 wrapper script 先把私钥复制到本地，再复制到容器。但这中方式仍然不安全，而且不优雅。

我们可以通过 docker 的 `-v` 命令行标志挂载数据卷。第一种办法就是简单粗暴地将整个 `~/.ssh` 目录挂载，但很难

```
docker run --name=foo-build -v ~/.ssh:/root/.ssh foo-build
```

该命令在 MacOS X 正常，但在 Linux 则不行。原因是 `~/.ssh/config` 文件的所有权。 `ssh` 期望该文件与当前用户拥有相同的所有权。容器内部，用户是 root 权限，但挂载的目录一般是普通的 Linux 用户的权限，即 developer。容器内如下：

```
$ ls ~/.ssh/config
-rw-r--r--  1 1000  1000  147 Jun  1 19:20 /root/.ssh/config
```

让 SSH 报错退出：
```
Bad owner or permissions on ~/.ssh/config
```

The solution is to mount only the key and workaround host checking later.

```
docker run --name=foo-build -v ~/.ssh/id_rsa:/root/.ssh/id_rsa foo-build
```
报错依然相同。但当带 `-t` 参数，返回如下：

```
$ docker run --name=foo-build -v ~/.ssh/id_rsa:/root/.ssh/id_rsa -t foo-build
The authenticity of host 'github.com (192.30.252.128)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```

当然，我们并不想手动与 SSH 实时交互。故而，我们需要找到一个强制执行的方式。有个称为 StrictHostChecking 的 SSH 客户端选项。

### StrictHostChecking
通常，你有按 `~/.ssh/known_hosts` 命名的文件，里面有已知主机的信息，但在容器内却没有。所以我们需要用客户端选项弃用这些检测。最简单的方式是将该选项放入 `~/.ssh/config` 文件（那个存在权限问题的文件）。

但是，我们只需要一个选项，故而在容器内直接创建该文件（create this file on the fly inside the container）是可行的。在 Dockerifle 内增加

```
RUN mkdir /root/.ssh && echo "StrictHostKeyChecking no " > /root/.ssh/config
```

重新执行 `docker run` ，这次会成功。

# 总结
最终的 Dockerfile：

```
FROM golang:1.6

RUN echo "[url \"git@github.com:\"]\n\tinsteadOf = https://github.com/" >> /root/.gitconfig
RUN mkdir /root/.ssh && echo "StrictHostKeyChecking no " > /root/.ssh/config
ADD .  /go/src/github.com/company/foo
CMD cd /go/src/github.com/company/foo && go get github.com/company/bar && go build -o /foo
```

构建步骤：
```
docker build -t foo-build .
docker run --name=foo-build -v ~/.ssh/id_rsa:/root/.ssh/id_rsa foo-build
docker cp foo-build:/foo foo
docker rm -f foo-build
docker rmi -f foo-build	
```

你可以将这些步骤加入 Makefile 或者定制构建脚本，在本地、CI 和其他任何地方安全使用。

SSH 私钥只有一次被复制到暂时的容器中用于构建，之后马上被移除。优雅安全的解决方式。