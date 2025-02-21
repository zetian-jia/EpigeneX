---
layout: default
title: flatpak
parent: Tutorials
nav_order: 8
---



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
