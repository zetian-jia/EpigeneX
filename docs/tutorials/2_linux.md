---
layout: default
title: linuxSystem
parent: Tutorials
nav_order: 2
---

# Linux System
{:.no_toc }
This is a tutorial for Linux system administration.

## Table of contents
{:.no_toc.text-delta }

- TOC
{:toc}

---

# Introduction

## Software Installation

### oh-my-zsh

different bash shells can be installed on Linux, but the most popular one is the Z shell (zsh). To install zsh and [oh-my-zsh](https://ohmyz.sh/), follow the steps below:

```bash
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

check version of zsh:
```bash
zsh --version
```

check shell bash or zsh:
```bash
echo $SHELL
```

the config file for zsh is located at `~/.zshrc`.

```shell
# Path to your Oh My Zsh installation.
export ZSH="$HOME/.oh-my-zsh"

plugins=(
    git
    zsh-autosuggestions
)

source $ZSH/oh-my-zsh.sh
```



# flatpak

[Flatpak](https://flathub.org/) is a technology that allows you to run applications in a sandboxed environment. It is similar to Snap or AppImage, but it is more lightweight and can run on any Linux distribution that supports it.

you should have root access to install flatpak.


## installation

[tutorial](https://flatpak.org/setup/Ubuntu)

```bash
sudo apt install flatpak
```

Install GNOME Software Flatpak plugin
The GNOME Software plugin makes it possible to install apps without needing the command line. To install, run:
```bash
sudo apt install gnome-software-plugin-flatpak
```

Add the Flathub repository

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```


## usage


### zotero

zotero is a popular reference manager.

```bash
flatpak install flathub org.zotero.Zotero

flatpak run org.zotero.Zotero
```

by the way, add xpi extension to zotero to open it in firefox.

[attanger](https://github.com/MuiseDestiny/zotero-attanger)
[better-bibtex](https://github.com/retorquere/zotero-better-bibtex)

### gimp

gimp is a popular image editor.

```bash
flatpak install flathub org.gimp.GIMP
```

### Typora

```bash
flatpak install flathub io.typora.Typora
```

### vscode
```bash
flatpak install flathub com.visualstudio.code
```

### tex

```bash
flatpak install flathub org.texstudio.TeXstudio
```



# crash

## gnome-extension ubuntu-appindicators@ubuntu.com crash
check crash
```bash
journalctl -b -1 -xe
```
```bash
3月 05 16:38:47 jiazetian-MS-7E06 gnome-shell[3293]: Unhandled promise rejection. To suppress this warning, add an error handler to your promise chain with .catch() or a try-catch block around your await expression. Stack trace of the failed promise:
                                                        refreshPropertyOnProxy@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/util.js:43:38
                                                        _translateNewSignals/<@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:212:18
                                                        _translateNewSignals@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:211:50
                                                        _onProxySignal/<@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:228:56
                                                        _onProxySignal@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:228:38
3月 05 16:38:47 jiazetian-MS-7E06 gnome-shell[3293]: Unhandled promise rejection. To suppress this warning, add an error handler to your promise chain with .catch() or a try-catch block around your await expression. Stack trace of the failed promise:
                                                        refreshPropertyOnProxy@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/util.js:43:38
                                                        _translateNewSignals/<@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:212:18
                                                        _translateNewSignals@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:211:50
                                                        _onProxySignal/<@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:228:56
                                                        _onProxySignal@/usr/share/gnome-shell/extensions/ubuntu-appindicators@ubuntu.com/appIndicator.js:228:38
```


从错误日志来看，ubuntu-appindicators@ubuntu.com 这个扩展很可能是导致问题的原因。

错误 "Unhandled promise rejection" 表示某个 JavaScript Promise 在执行过程中失败了，但没有被 .catch() 处理，这可能导致 GNOME Shell 崩溃或无响应。

错误堆栈指向 appIndicator.js 和 util.js，这些文件属于 ubuntu-appindicators 扩展，所以这个扩展极有可能是问题来源。
如何排查和解决？
```bash
gnome-extensions disable ubuntu-appindicators@ubuntu.com

```
然后尝试重启 GNOME Shell：
```bash
Alt + F2，输入 `r`，然后回车  （Wayland 下无效，需要重启）
```
或者 直接重启计算机

如果问题持续，可以卸载 ubuntu-appindicators
```bash
gnome-extensions uninstall ubuntu-appindicators@ubuntu.com

```
启用扩展：
```bash
gnome-extensions enable ubuntu-appindicators@ubuntu.com

```
禁用扩展：
```bash
gnome-extensions disable ubuntu-appindicators@ubuntu.com

```
如果你想查看所有扩展（包括禁用的）
```bash 
gnome-extensions list
```

查看当前启用的 GNOME 扩展
```
gnome-extensions list --enabled
```



