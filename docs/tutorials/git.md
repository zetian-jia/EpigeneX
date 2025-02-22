---
layout: default
title: git
parent: Tutorials
nav_order: 5
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


