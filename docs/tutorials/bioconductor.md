---
layout: default
title: Bioconductor
parent: Tutorials
nav_order: 10
---

# Running the Container

### Docker

[Reference](https://github.com/Bioconductor/bioconductor_docker)

#### Step 1: Install Docker

First, you need to install Docker on your machine. You can download it from [here](https://www.docker.com/products/docker-desktop) (for Windows or Mac), or use the following command for Linux (example: for Clash proxy setup):

```bash
./clash-linux

# Set proxy (if you are using a proxy server)
export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"
```


Step 2: Pull the Bioconductor Docker Image
To download the Bioconductor Docker image, run the following command:

```bash
docker pull bioconductor/bioconductor_docker:devel
```

Step 3: Run RStudio Server in Docker
To start RStudio Server in the container, use this command:

```bash
docker run -e PASSWORD=123 \
	-p 8787:8787 \
	bioconductor/bioconductor_docker:devel
```
Once the container is running, you can access RStudio Server at:

```bash
http://localhost:8787
```

Username: rstudio
Password: 123



Step 4: Run R from the Command Line
To run R directly from the command line inside the Docker container, use:


```bash
docker run -it --user rstudio bioconductor/bioconductor_docker:devel R
```

Step 5: Mount Volumes with -v
You can mount a local directory to the Docker container using the -v flag. This is useful for sharing data between your local machine and the container.

For example, to mount your local R library directory:

`-v`将附加卷安装到 docker 映像

以交互方式运行它
```bash
docker run \
    -v /home/jiazet/R/rstudio/4.5.1:/usr/local/lib/R/host-site-library \
    -it \
    --user rstudio \
    bioconductor/bioconductor_docker:devel bash
```

使用 RStudio 界面运行
```bash
docker run \
  	--network host \
	-v /home/jiazet/R/rstudio/4.5.1:/usr/local/lib/R/host-site-library \
  	-e PASSWORD=123 \
  	-p 8787:8787 \
  	bioconductor/bioconductor_docker:devel
```
Docker 容器默认在退出后删除容器内的所有数据，除非你明确地将数据保存到宿主机或持久化卷

--network host：使用宿主机的网络模式，这通常用于容器和宿主机共享网络资源


When to Use Docker Compose?
Docker Compose is a useful tool when you need to share data between multiple containers or run multi-container applications. It allows you to define and manage multi-container Docker applications with ease.

Example: `docker-compose.yml` Configuration
You can define your services, volumes, environment variables, and ports in a docker-compose.yml file.

Here’s an example configuration for running Bioconductor with RStudio Server:

```bash
version: "3.8"
services:
  bioconductor:
    image: bioconductor/bioconductor_docker:devel
    volumes:
      - /home/jiazet/R/rstudio/4.5.1:/usr/local/lib/R/host-site-library
    environment:
      - PASSWORD=password
    ports:
      - "8787:8787"

```

To run in the background, use the -d or --detach flag, `-d`
```bash
docker-compose up
```

### Singularity

[reference](https://rocker-project.org/use/singularity.html) and [bioconductor docker](https://github.com/Bioconductor/bioconductor_docker)

```bash
./clash-linux

export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"
```

1. Importing a Rocker Image
```bash
singularity pull docker://rocker/rstudio:4.4.2
```

2. Running a Rocker Singularity container (localhost, no password)

```bash
singularity exec \
   --scratch /run,/var/lib/rstudio-server \
   --workdir $(mktemp -d) \
   rstudio_4.4.2.sif \
   rserver --www-address=127.0.0.1 --server-user=$(whoami)
```

open `127.0.0.1:8787`

3. Running a Rocker Singularity container with password authentication
To enable password authentication, set the PASSWORD environment variable and add the --auth-none=0 --auth-pam-helper-path=pam-helper options:

```bash
PASSWORD='123' singularity exec \
   --scratch /run,/var/lib/rstudio-server \
   rstudio_4.4.2.sif \
   rserver --auth-none=0 --auth-pam-helper-path=pam-helper --server-user=$(whoami)
```

`http://hostname:8787`

local user ID

PASSWORD: 123

4. Additional Options for RStudio >= 1.3.x

In addition, RStudio >= 1.3.x enforces a stricter policy for session timeout, defaulting to 60 Minutes. You can opt in to the legacy behaviour by adding the following parameters:

`'auth-pam-requires-priv' is deprecated and will be discarded.`
```bash
--session-timeout-minutes=0 --auth-timeout-minutes=0
```

