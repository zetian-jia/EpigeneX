---
layout: default
title: apptainer
parent: Tutorials
nav_order: 13
---





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