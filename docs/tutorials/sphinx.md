---
layout: default
title: Sphinx
parent: Tutorials
nav_order: 3
---

# Sphinx
{. no_toc}

[Sphinx](https://www.sphinx-doc.org/en/master/usage/installation.html#conda-package) is a tool that makes it easy to create intelligent and beautiful documentation, written by <NAME> and licensed under the BSD license. It was originally created for the Python documentation, but it has since been extended to support other languages and domains.

出色的工具，可用于各种语言的软件项目文档的自动生成。

- 输出格式: HTML（包括Windows HTML帮助），LaTeX（适用于可打印的PDF版本），ePub，Texinfo，手册页，纯文本

## Table of contents
{:.no_toc.text-delta }

- TOC
{:toc}

---

## Installation

installation via conda:

```bash
conda create -n sphinx python=3.10 sphinx sphinx_rtd_theme
```

activate the environment:

```bash
conda activate sphinx
```

## getting started

Sphinx 纯文本文档源集合的根目录称为源目录。此目录还包含 Sphinx 配置文件conf.py

Sphinx 附带一个名为sphinx-quickstart的脚本，该脚本会设置源目录

```bash

cd methylTracer_docs/

sphinx-quickstart

```

{: .info-title }
> reStructuredText
>
> toctree是 reStructuredText指令，一种用途非常广泛的标记

`build/`: 包含生成的文档的目录。
`make.bat` and `Makefile`: 便捷脚本
`source/conf.py`: 保存 Sphinx 项目配置的 Python 脚本
`source/index.rst`: 项目的主页，也是项目文档的入口


Option-1: make html:

```bash

make html
```

Option-2: build html:
```bash
sphinx-build -M html source build
```


open `_build/html/index.html` in your browser to view the generated documentation.


clean up:

```bash

make clean

```


## github pages

To publish your Sphinx documentation on GitHub Pages, follow these steps:

donot load `build/` folder in your repository.