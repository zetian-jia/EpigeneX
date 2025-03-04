---
layout: default
title: containerization
parent: Tutorials
nav_order: 3
---


# containerization
{:.no_toc }
best practices for containerization

# Table of contents
{:.no_toc.text-delta }

- TOC
{:toc}

---

# docker

## install (ubuntu)

[install tutorial](https://docs.docker.com/engine/install/)

clean up old versions
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

卸载 Docker Engine
1. 卸载 Docker Engine、CLI、containerd 和 Docker Compose 软件包
```bash
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```
2. 主机上的镜像、容器、卷或自定义配置文件不会自动删除。要删除所有镜像、容器和卷，请执行以下操作
```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```
3. 删除源列表和密钥环
```bash
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.asc
```


1. 设置 Docker 的apt存储库

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. 安装 Docker 包

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. 通过运行镜像来验证安装是否成功
```bash
sudo docker run hello-world
```

## Linux 系统中通常需要执行一些额外的配置步骤

**Docker 守护进程（Docker Daemon）**: 是负责执行所有 Docker 容器操作的后台服务。它默认以 root 用户身份运行，通常以服务的形式在系统启动时启动。

{: .info-title}
> **Note**
>
> Docker 守护进程与外界通信的接口是一个 Unix 套接字，默认位置为 `/var/run/docker.sock`，而不是 TCP 网络端口。这个 Unix 套接字由 Docker 守护进程创建，并且它是 root 用户所有的


### 如果你不想每次使用 docker 命令时都加上 sudo，可以采取以下步骤

创建 Docker 组：在 Linux 系统中创建一个名为 docker 的用户组，并将需要使用 Docker 的用户添加到这个组中。这样，Docker 守护进程将允许 docker 组的成员访问其 Unix 套接字，而无需使用 sudo

步骤：
创建 docker 组（如果系统中没有的话）
```bash
sudo groupadd docker
```
将当前用户添加到 docker 组：

-a 选项：表示 append（追加）的意思
-G 选项：表示要将用户添加到的组名

```bash
sudo usermod -aG docker $USER
```
重新登录：用户添加到新组后，必须重新登录才能使更改生效

- 如何检查是否存在 docker 组
```bash
grep docker /etc/group
```

配置 json-file 日志驱动程序以启用日志轮转防止单个日志文件占用过多磁盘空间。

编辑 `/etc/docker/daemon.json` 文件

```bash
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}

```

每个日志文件的最大大小为 10MB，最多保存 3 个日志文件。


## 将代理服务器与 Docker CLI 结合使用

donot use docker desktop

### 配置 Docker 客户端
您可以使用位于 的 JSON 配置文件为 Docker 客户端添加代理配置`~/.docker/config.json`

您所配置的代理环境变量 只会影响到容器内部的网络访问，而不会直接影响到 Docker 命令行工具（CLI）或 Docker 引擎（Daemon）的代理设置

当你设置环境变量如 HTTP_PROXY 或 HTTPS_PROXY 时，这些**设置会应用到你运行的 Docker 容器中**。这意味着：

容器内的进程将通过代理服务器来访问网络资源。

```bash
{
 "proxies": {
   "default": {
     "httpProxy": "http://twb.show.101",
     "httpsProxy": "http://twb.show.01",
     "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
   }
 }
}
```
这些设置仅用于配置容器的代理环境变量，不用作 Docker CLI 或 Docker Engine 本身的代理设置。

`--rm`：容器在退出时自动删除
些代理环境变量通常用于容器内的应用程序和服务来配置它们如何通过代理访问外部网络。
```bash
(base) ➜  bioconductor_compose git:(main) ✗ docker run --rm alpine sh -c 'env | grep -i  _PROXY'
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
f18232174bc9: Pull complete 
Digest: sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
Status: Downloaded newer image for alpine:latest
HTTPS_PROXY=http://twb.sho1
no_proxy=*.test.example.com,.example.org,127.0.0.0/8
NO_PROXY=*.test.example.com,.example.org,127.0.0.0/8
https_proxy=http://twb.showm801
http_proxy=http://twb.showmyip01
HTTP_PROXY=http://twb.showmy01
```

{: .info-title}
> v2free
>
> copy proxy from [v2free](https://v2free.org/user/node)

### Docker CLI 和 Docker Engine 的代理设置
而 Docker CLI 和 Docker Daemon 本身则是需要单独配置代理的。CLI 是您在命令行中运行的 Docker 工具，Daemon 是 Docker 引擎本身，它负责管理和启动容器。
Docker Daemon 代理：如果 Docker 引擎本身需要通过代理服务器连接到网络

### Docker CLI 配置文件（config.json）属性

`~/.docker/config.json`

demo

```json

{
  "HttpHeaders": {
    "MyHeader": "MyValue"
  },
  "psFormat": "table {{.ID}}\\t{{.Image}}\\t{{.Command}}\\t{{.Labels}}",
  "imagesFormat": "table {{.ID}}\\t{{.Repository}}\\t{{.Tag}}\\t{{.CreatedAt}}",
  "pluginsFormat": "table {{.ID}}\t{{.Name}}\t{{.Enabled}}",
  "statsFormat": "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}",
  "servicesFormat": "table {{.ID}}\t{{.Name}}\t{{.Mode}}",
  "secretFormat": "table {{.ID}}\t{{.Name}}\t{{.CreatedAt}}\t{{.UpdatedAt}}",
  "configFormat": "table {{.ID}}\t{{.Name}}\t{{.CreatedAt}}\t{{.UpdatedAt}}",
  "serviceInspectFormat": "pretty",
  "nodesFormat": "table {{.ID}}\t{{.Hostname}}\t{{.Availability}}",
  "detachKeys": "ctrl-e,e",
  "credsStore": "secretservice",
  "credHelpers": {
    "awesomereg.example.org": "hip-star",
    "unicorn.example.com": "vcbait"
  },
  "plugins": {
    "plugin1": {
      "option": "value"
    },
    "plugin2": {
      "anotheroption": "anothervalue",
      "athirdoption": "athirdvalue"
    }
  },
  "proxies": {
    "default": {
      "httpProxy":  "http://user:pass@example.com:3128",
      "httpsProxy": "https://my-proxy.example.com:3129",
      "noProxy":    "intra.mycorp.example.com",
      "ftpProxy":   "http://user:pass@example.com:3128",
      "allProxy":   "socks://example.com:1234"
    },
    "https://manager1.mycorp.example.com:2377": {
      "httpProxy":  "http://user:pass@example.com:3128",
      "httpsProxy": "https://my-proxy.example.com:3129"
    }
  }
}
```

### 指定守护进程主机（-H、--host）
如果您未指定-H标志，并且未使用自定义 上下文，则命令将使用以下默认套接字：

`unix:///var/run/docker.sock在 macOS 和 Linux 上`

使用 TCP 套接字
以下示例显示如何通过 TCP 调用IP 地址为、监听端口docker ps的远程守护进程：174.17.0.12376
```bash
docker -H tcp://174.17.0.1:2376 ps

docker -H ssh://jiazetian@57 ps
```
按照惯例，Docker 守护进程使用端口2376进行安全的 TLS 连接，并使用端口2375进行不安全的非 TLS 连接

### 守护进程代理配置

{: .info-title}
> **Note**
>
> daemon.jsonDocker Desktop 会忽略 中指定的代理配置。如果您使用 Docker Desktop，则可以使用Docker Desktop 设置配置代理。

在 `~/.docker/` 目录下的 daemon.json 文件是不能直接用来配置 Docker 守护进程的设置的，因为这个文件专门用于存储 Docker CLI 的配置，而不是 Docker Daemon（守护进程）的配置

全局配置文件路径: Docker 守护进程的配置文件通常位于 `/etc/docker/daemon.json`

### 守护进程配置文件
`/etc/docker/daemon.json`:这才是 Docker 守护进程在启动时读取的配置文件。

更改配置文件后，重新启动守护进程以使代理配置生效：
```bash
sudo systemctl restart docker
```
如果您将 Docker 守护程序作为 systemd 服务运行，则可以创建一个 systemd 插件文件来设置该docker服务的变量。
1. 为该服务创建一个 systemd 插入目录docker
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```
2. 创建一个名为的文件`/etc/systemd/system/docker.service.d/http-proxy.conf`并添加HTTP_PROXY环境变量

```bash
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
```

3.如果您有需要在没有代理的情况下联系的内部 Docker 注册表，则可以通过NO_PROXY环境变量指定它们
该NO_PROXY变量指定一个字符串，其中包含应从代理中排除的主机的逗号分隔值。您可以指定以下选项来排除主机：

```bash
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,docker-registry.example.com,.corp"
```

4. 刷新更改并重新启动 Docker
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
5. 验证配置是否已加载并与您所做的更改相匹配，例如：
```bash
sudo systemctl show --property=Environment docker

Environment=HTTP_PROXY=http://127.0.0.1:7890 HTTPS_PROXY=http://127.0.0.1:7890 NO_PROXY=localhost,docker-registry.example.com,.corp
```

6. set proxy for docker cli (V2free)

启动clash【切记：不要加 sudo】
```bash
./clash-linux

export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"
```


### 环境变量

### 配置文件
默认情况下，Docker 命令行将其配置文件存储在目录`.docker`内名为的目录中$HOME

### 更改.docker目录
要指定不同的目录，请使用DOCKER_CONFIG 环境变量或--config命令行选项
```bash
docker --config ~/testconfigs/ ps
```


## TIPS


1. docker desktop and docker cli

if use docker desktop the proxy will be change! so do not use docker desktop and docker cli at the same time.

2. docker images and container

镜像是一个包含应用程序及其依赖项的只读模板，通常用于创建容器。容器是镜像的一个实例，运行中的应用。




# apptainer

## SetUID

[Linux SetUID（SUID）文件特殊权限用法详解](https://c.biancheng.net/view/868.html)

除了 rwx 权限，还会用到 s 权限, 此种权限通常称为 SetUID，简称 SUID 特殊权限

功能是，只要用户对设有 SUID 的文件有执行权限，那么当用户执行此文件时，会以文件所有者的身份去执行此文件，一旦文件执行结束，身份的切换也随之消失。

其他人对此文件也有执行权限，这就意味着，任何一个用户都可以用文件所有者，也就是 root 的身份去执行 passwd 命令。
```bash
[root@localhost ~]# ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 22984 Jan  7  2007 /usr/bin/passwd
```


Linux 系统中，绝对多数命令的文件所有者默认都是 root。

普通用户可以使用 cat 命令查看 /etc/shadow 文件吗？答案的否定的，因为 cat 不具有 SUID 权限，因此普通用户在执行 cat /etc/shadow 命令时，无法以  root 的身份，只能以普通用户的身份，因此无法成功读取。


由此，我们可以总结出，SUID 特殊权限具有如下特点：

- 只有可执行文件才能设定 SetUID 权限，对目录设定 SUID，是无效的。
- 用户要对该文件拥有 x（执行）权限。
- 用户在执行该文件时，会以文件所有者的身份执行。
- SetUID 权限只在文件执行过程中有效，一旦执行完毕，身份的切换也随之消失。

您好，Singularity 是一个独立的二进制文件（或者至少是一组独立的二进制文件）。[但它需要 SUID 位才能正常工作](https://github.com/apptainer/singularity/issues/




## 用户命名空间和 Fakeroot

[Singularity 如果不适用 SetUID，那它通过普通用户安装运行是要求开启](https://blog.weiyan.cc/tech/try-singularity/) User [Namespace](https://docs.sylabs.io/guides/3.8/admin-guide/user_namespace.html)

用户命名空间是一种隔离功能，允许进程在该命名空间内以不同于外部允许的用户标识符和/或权限运行。用户可能在用户命名空间之外的系统上拥有uid权限 1001，但在命名空间内以不同的权限运行程序uid。


SingularityCE 在 3 种情况下使用用户命名空间：

1. 当setuid工作流程被禁用或 SingularityCE 未在 root 权限下安装时。

2. 当使用该选项运行容器时--userns。

3. 当--fakeroot构建或运行容器时，用于模拟 root 用户。

### 用户命名空间要求
为了允许非特权创建用户命名空间，需要内核 >=3.8，由于用户命名空间的安全修复，建议使用 >=3.18（3.18 还增加了 Singularity 使用的 OverlayFS 支持）。

Debian/Ubuntu 系统默认内核版本较低，需要手动升级内核。
```bash
sudo sh -c 'echo kernel.unprivileged_userns_clone=1 \
    >/etc/sysctl.d/90-unprivileged_userns.conf'
sudo sysctl -p /etc/sysctl.d /etc/sysctl.d/90-unprivileged_userns.conf
```

RHEL/CentOS 7
从 7.4 开始，包含内核支持，但必须启用
```bash
sudo sh -c 'echo user.max_user_namespaces=15000 \
    >/etc/sysctl.d/90-max_net_namespaces.conf'
sudo sysctl -p /etc/sysctl.d /etc/sysctl.d/90-max_net_namespaces.conf
```

使用 uname 命令查看内核信息

```bash
$ uname -r                  
4.18.0-240.22.1.el8_3.x86_64
```

OR 使用 `cat /etc/os-release` 命令查看内核信息

```bash
cat /etc/os-release

NAME="CentOS Linux"
VERSION="8"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="CentOS Linux 8"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:8"
HOME_URL="https://centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"
CENTOS_MANTISBT_PROJECT="CentOS-8"
CENTOS_MANTISBT_PROJECT_VERSION="8"
```
### 非特权安装

```bash
conda
```

### Fakeroot 功能
Fakeroot（或通常称为无根模式）允许非特权用户通过利用具有用户命名空间 UID/GID 映射的用户命名空间以“假根”用户身份运行容器。

除了用户命名空间支持之外，SingularityCE 还必须操作 subuid和subgid映射它所创建的用户命名空间。

基础知识

Fakeroot 依靠/etc/subuid和/etc/subgid文件来查找从真实用户和组 ID 到主机系统上每个用户的一系列空置 ID 的配置映射，这些 ID 可以在用户名空间中重新映射。用户必须在这些系统配置文件中有一个条目才能使用 fakeroot 功能。


对于用户来说，foo输入`/etc/subuid`可能是：
```bash
foo:100000:65536
```
其中foo，是用户名、是可在用户命名空间 uid 映射中 100000使用的 UID 范围的起始，以及可用于映射的 UID 数量。foo65536

相同`/etc/subgid`
```bash
foo:100000:65536
```

如果您想添加另一个用户bar，/etc/subuid将 /etc/subgid如下所示
```bash
foo:100000:65536
bar:165536:65536
```

| 用户   | 主机 UID | 子 UID/GID 范围      |
|--------|----------|----------------------|
| 富     | 1000     | 100000 至 165535     |
| 酒吧   | 1001     | 165536 至 231071     |


文件系统注意事项

添加 fakeroot 映射
使用选项创建新的映射条目，以便可以使用 Singularity 的 fakeroot 功能

用户命名空间 UID/GID 映射

UID（User Identifier）是操作系统中用于唯一标识用户的数字。每个用户在系统中都有一个唯一的 UID，用于区分不同的用户。

UID（User Identifier）：表示用户的标识符。

GID（Group Identifier）：表示用户所属组的标识符

UID 0：通常是超级用户（root 用户）的 UID
