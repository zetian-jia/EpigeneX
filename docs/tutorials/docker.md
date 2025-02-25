---
layout: default
title: docker
parent: Tutorials
nav_order: 9
---

# docker

best practices for docker



## install (ubbuntu)

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
     "httpProxy": "http://twb.showmyip.top:11801",
     "httpsProxy": "http://twb.showmyip.top:11801",
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
HTTPS_PROXY=http://twb.showmyip.top:11801
no_proxy=*.test.example.com,.example.org,127.0.0.0/8
NO_PROXY=*.test.example.com,.example.org,127.0.0.0/8
https_proxy=http://twb.showmyip.top:11801
http_proxy=http://twb.showmyip.top:11801
HTTP_PROXY=http://twb.showmyip.top:11801
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

docker -H ssh://jiazetian@210.42.120.57 ps
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

BINGO
```
(base) ➜  bioconductor_compose git:(main) ✗ singularity pull docker://rocker/rstudio:4.4.2   
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob 4927158e7cb8 done   | 
Copying blob 98b4444c3e59 done   | 
Copying blob 5fa970fe3a02 done   | 
Copying blob 5a7813e071bf done   | 
Copying blob 619d6b2dbf39 done   | 
Copying blob c431a6837342 done   | 
Copying blob a0d63a8d22d5 done   | 
Copying blob d8833c1fcb77 done   | 
Copying blob 8d545f1f0f82 done   | 
Copying blob a0f19b4e324e done   | 
Copying blob 1d1f3f67e298 done   | 
Copying blob 8a85ff299c33 done   | 
Copying blob a2b7b22245e9 done   | 
Copying blob 9bf9bf0e922b done   | 
Copying blob 12381b307ecb done   | 
Copying blob 2617b2670e16 done   | 
Copying blob dee49bb49d26 done   | 
Copying blob e6bb380d519e done   | 
Copying config 5669edbdbc done   | 
Writing manifest to image destination
2025/02/26 03:04:51  info unpack layer: sha256:5a7813e071bfadf18aaa6ca8318be4824a9b6297b3240f2cc84c1db6f4113040
2025/02/26 03:04:51  info unpack layer: sha256:619d6b2dbf391df5086ca4150c1d560840f2d136f84e0cb1a7eaa47730a0b678
2025/02/26 03:04:51  info unpack layer: sha256:98b4444c3e59cf844097fd83f23c5871eefa7192c9bdfb16360819d44cf41d62
2025/02/26 03:04:54  info unpack layer: sha256:5fa970fe3a02f44f436da8f9bff5221ecd4beae0972419070fe27d6b076872fc
2025/02/26 03:04:54  info unpack layer: sha256:4927158e7cb86741814c82d851c8450bdf71f5df6e9677d9e85a7f18865d3610
2025/02/26 03:04:54  info unpack layer: sha256:c431a6837342ec8c4aedf8e4dc2f2f123dbc996404347b63799cb113649708df
2025/02/26 03:04:54  info unpack layer: sha256:a0d63a8d22d514a030c5b445f408a34f87a91d30975dde76c2365111de78c74c
2025/02/26 03:04:54  info unpack layer: sha256:d8833c1fcb77e92d3c3fe41ed915f82bc158e05aa196543a4bc9e8db0d09dfa8
2025/02/26 03:04:54  info unpack layer: sha256:8d545f1f0f8201a2019dbd222715d72331df3c9e3a8bdd3c7d8bbee97b3290fc
2025/02/26 03:04:54  info unpack layer: sha256:a0f19b4e324ef03f31321b1082dcc44ab69eb730accc0817e611463603fcaa21
2025/02/26 03:04:54  info unpack layer: sha256:1d1f3f67e298288fa5cc42b6f6b3ae69a18f00176596059820a9ef113010b05b
2025/02/26 03:04:54  info unpack layer: sha256:8a85ff299c334b2ad91ee0b794d9e40af87e745516fc09973aa0c1b3ea74deac
2025/02/26 03:04:54  info unpack layer: sha256:a2b7b22245e9c5d213ced9b9ec39e3c15b9283c3be90db72f577435ff4816bd1
2025/02/26 03:04:58  info unpack layer: sha256:9bf9bf0e922b72297a41052aa9821591823e6119c6d7b9d0469bea687be47335
2025/02/26 03:04:58  info unpack layer: sha256:12381b307ecb335a95180ad7bfef901e8ec72f64aa5d9fe5d45eafe2185cdd73
2025/02/26 03:04:58  info unpack layer: sha256:2617b2670e163d44a2c0cadb57e5dd7807d99ec7f2c9a8280fe5f584039cf3a3
2025/02/26 03:04:58  info unpack layer: sha256:dee49bb49d2645d4cd4f41256dd81d2fa48d3b6bbdf118237eaf8abbf7ef7acc
2025/02/26 03:04:58  info unpack layer: sha256:e6bb380d519e13fe619ab9ea885d15b38233dc4b25e3fd85391e6b84d6c3a518
INFO:    Creating SIF file...
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



