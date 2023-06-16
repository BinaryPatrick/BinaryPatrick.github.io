---
layout: post
title: 'LXC: Installing Docker on a Debian CT'
date: 2023-1-1 12:00:00 -0500
category: 'Service Setup'
tags: ['proxmox', 'lxc', 'debian', 'docker']
---

A quick guide to getting docker running on a Debian CT

<!--more-->

## CT Configuration

First make sure your container is running in privledged mode and nested is enabled in Options > Features

![features](/assets/img/lxc-docker-setup-1.png)

## Installing Docker

Then you'll need to login and install docker.

```bash
sudo apt install docker.io -y && sudo systemctl enable docker
```

Then start and confirm the service

```bash
sudo systemctl start docker && sudo systemctl status docker
```

To ensure Docker is running correctly you can try to run a simple hello-world container

```bash
sudo docker run hello-world
```

## Install Docker Compose

Debian unfortunately only ships with docker compose v1. The plugin for v2 needs to be installed manually from github releases. Keep in mind the version number can be whatever is the latest (v2.14.2 as of this post).

```bash
mkdir -p ~/.docker/cli-plugins
```

```bash
curl -sSL https://github.com/docker/compose/releases/download/v2.14.2/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
```

```bash
chmod +x ~/.docker/cli-plugins/docker-compose
```

Now check to make sure v2 is installed

```bash
docker compose version
```

## Run Docker from a non-root user without sudo

```bash
sudo usermod -aG docker $USER
```

> You'll need to logout and log back in for the change to take effect
{: .prompt-warning }


## Installing Lazydocker

Lazydocker is basically a CLI portainer. Rather than running a service for a single container running in LXC. I like to install lazy docker to manage things. Install is easy by using the script they provide, but I prefer to run my own.

```bash
#!/bin/bash

# allow specifying different destination directory
DIR="/usr/local/bin"

# map different architecture variations to the available binaries
ARCH=$(uname -m)
case $ARCH in
    i386|i686) ARCH=x86 ;;
    armv6*) ARCH=armv6 ;;
    armv7*) ARCH=armv7 ;;
    aarch64*) ARCH=arm64 ;;
esac

# prepare the download URL
GITHUB_LATEST_VERSION=$(curl -L -s -H 'Accept: application/json' https://github.com/jesseduffield/lazydocker/releases/latest | sed -e 's/.*"tag_name":"\([^"]*\)".*/\1/')
GITHUB_FILE="lazydocker_${GITHUB_LATEST_VERSION//v/}_$(uname -s)_${ARCH}.tar.gz"
GITHUB_URL="https://github.com/jesseduffield/lazydocker/releases/download/${GITHUB_LATEST_VERSION}/${GITHUB_FILE}"

# install/update the local binary
curl -L -o lazydocker.tar.gz $GITHUB_URL
tar xzvf lazydocker.tar.gz lazydocker
install -Dm 755 lazydocker -t "$DIR"
rm lazydocker lazydocker.tar.gz
```

Now you can simply run it with the following command.

```bash
lazydocker
```