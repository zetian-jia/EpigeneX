---
layout: default
title: conda
parent: Tutorials
nav_order: 2
---

# conda
{: .no_toc }

conda is the basic package manager for bioinformatics software. It is widely used in the field of bioinformatics and provides a convenient way to install and manage software packages. In this tutorial, we will show you how to install and use conda on your local machine.

## Table of contents
{: .no_toc.text-delta }

- TOC
{:toc}

## miniconda

The first step is to download and install [miniconda](https://conda.io/miniconda.html), which is a minimal installation of conda. Miniconda is a small, bootstrap version of conda that includes only conda, Python, the conda package manager, and a few essential packages like pip, zlib, and libffi.

{: .info-title }
> **Note**
>
> If you have already installed conda on your machine, you can skip this step.

### STEP 1: Download and install miniconda

Download the latest version of miniconda from the website and install it on your machine. You can follow the instructions on the website to complete the installation.

```bash
wget -c https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

install miniconda:
```bash
bash Miniconda3-latest-Linux-x86_64.sh
```

### STEP 2: add channels

After installing miniconda, you need to add channels to the conda package manager. Channels are repositories of software packages that are available for installation. You can add the following channels to your conda installation:

```bash
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/cloud/bioconda/
conda config --set show_channel_urls yes
```

check the channels:
```bash
conda config --get channels
conda config --show channels
```

## Using conda

Now that you have installed and added the necessary channels, you can use conda to install and manage software packages.

### STEP-1: Create a new environment

to create a new environment, use the following command:

{: .info-title }
> **methylTracer**
>
> [methylTracer](https://github.com/zetian-jia/methylTracer) is a software package for analysis DNA methylation data.

```bash
conda create -n methylTracer r-base=4.3.1
```

### STEP-2: Activate the environment

To activate the environment, use the following command:

```bash
conda activate methylTracer
```

deactivate the environment:

```bash
conda deactivate
```

### STEP-3: Install packages

To install a package, use the following command:

```bash
conda install bedtools
```

if you want to install a specific version of a package, use the following command:

```bash
conda install bedtools=2.30.0
```

### STEP-4: Check conda environment

To check the conda environment, use the following command:

```bash
conda env list

# or
conda info --envs
```

## advanced conda
