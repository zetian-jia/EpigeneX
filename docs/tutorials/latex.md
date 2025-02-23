---
layout: default
title: latex
parent: Tutorials
nav_order: 8
---




# latex

latex is a typesetting language used to create high-quality documents. It is widely used in scientific and technical writing, as well as in the publishing industry. Here are some tips and tricks for using latex effectively.

install [latex](https://www.tug.org/texlive/quickinstall.html) in ubuntu


## install steps

working directory of your choice:
```bash
cd /tmp 
```

Download:
```bash
wget https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz
```

note final - on that command line
```bash
zcat < install-tl-unx.tar.gz | tar xf -
```

cd to the directory:
```bash
cd install-tl-*
```

may take **several hours** to run (8 GB)
```bash
sudo perl ./install-tl --no-interaction
```

path to the bin directory:

or write it to ~/.bashrc or ~/.zshrc
```bash
export PATH=/usr/local/texlive/2024/bin/x86_64-linux:$PATH

```

