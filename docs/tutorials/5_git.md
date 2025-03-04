---
layout: default
title: git
parent: Tutorials
nav_order: 5
---

# git
{:.no_toc }
best practices for git

# Table of contents
{:.no_toc.text-delta }

- TOC
{:toc}

---

# git

## example: sphinx

you create a new repository on GitHub, and you have a new local directory, How to push local to remote repository?

I count this in sphinx project.

before push, you should setup github ssh config by using keygen.

### STEP-1: initial local

cd to local directory

```bash
git init
```

set-ulr to remote repository. this step need to set ssh key first.

```bash
git remote set-url origin git@hotmail:zetian-jia/methylTracer_docs.git
```

if local branch is master, you should rename it to main.

```bash
git branch -m master main
```

### STEP-2: merge local and remote

because I create remote repository with README and License, so I should merge local and remote before push.

I like `git pull`, instand of `git fetch` and `git merge`.

```bash
git pull origin main --allow-unrelated-histories --no-rebase
```

add all files to commit

```bash
git add .
```

commit changes

```bash
git commit -m "init commit sphinx"
```

### STEP-3: push local to remote repository

push local to remote repository

```bash
git push origin main
```

## BUG

### Bug1

```bash
kex_exchange_identification: Connection closed by remote host
```

出现这个bug的背景：在mac等设备使用clash verge软件会自动走7989端口代理，但是通过命令行`git`会走Port 22端口，因此，要么更改git端口，要么更改clash为直连。这里推荐更改git端口为443可以解决问题。

第一步：正常使用clash软件打开代理

第二步：修改`~/.ssh/config`

```bash
host hotmailgit
  Hostname ssh.github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/ubuntu_hotmail_ed25519
  User git
  Port 443
```

第三步：：可以终端正常使用git
```bash
git remote set-url origin git@hotmailgit:zetian-jia/methylTracer.git
git push
```

关于上述问题，需要阅读github [ISSUE](https://github.com/vernesong/OpenClash/issues/1960),官方文档解决git [SSH错误](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)

Using SSH over the HTTPS port
有时，防火墙会完全拒绝SSH连接。如果无法使用带凭据缓存的HTTPS克隆，则可以尝试使用通过HTTPS端口建立的SSH连接进行克隆。大多数防火墙规则应该允许这样做，但代理服务器可能会干扰。
To test if SSH over the HTTPS port is possible, run this SSH command:

```bash
$ ssh -T -p 443 git@ssh.github.com
# Hi USERNAME! You've successfully authenticated, but GitHub does not
# provide shell access.
```
**NOTE**

The hostname for port 443 is ssh.github.com, not github.com.

Now, to clone the repository, you can run the following command:
```bash
git clone ssh://git@ssh.github.com:443/YOUR-USERNAME/YOUR-REPOSITORY.git
```

在 SSH 配置文件（通常位于 `~/.ssh/config`）中，Host 的名字对大小写不敏感。

如果您能够通过端口443 SSH进入git@ssh.github.com，则可以覆盖SSH设置，以强制通过该服务器和端口运行到GitHub.com的任何连接。


### Bug2

```bash
fatal: The current branch main has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin main
```

because you have not set the remote as upstream, you should use the command:

```bash
git push --set-upstream origin main
```



