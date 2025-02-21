---
layout: default
title: linuxSystem
parent: Tutorials
nav_order: 1
---

# Linux System
{:.no_toc }
This is a tutorial for Linux system administration.

## Table of contents
{:.no_toc.text-delta }

- TOC
{:toc}

---

## Introduction








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



