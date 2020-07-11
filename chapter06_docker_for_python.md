---
description: 'https://qiita.com/SLEAZOIDS/items/a2272b02e9a964596198'
---

# Chapter06 Docker For Python

### はじめに

ここでは、Pythonの分析環境としてJupyter lab環境を構築するための方法をまとめておきます。Anacondaを利用する方法です。

### Dockerファイル

まずはDockerファイルを書いていきます。ビルドコンテキストとして、`python_docker`というディレクトリを用意します。あわせてマウントするようのディレクトリ`Python_mounted_dir`を作ります。

```text
➜ mkdir python_docker
➜ mkdir Python_mounted_dir

➜ cd python_docker
➜ touch Dockerfile
```

基本的な流れはubuntuのイメージを使って、Anacondaアーカイブからshファイルを取得して、それをバッチモードでインストールすることで環境を構築します。また、Pythonのライブラリもインストールしておきます。

```text
# Base image
FROM ubuntu:18.04

# Update
RUN set -x && \
    apt update && \
    apt upgrade -y

# Install util
RUN set -x && \
    apt install -y wget && \
    apt install -y sudo

# Install anaconda
# -b : batch mode
RUN set -x && \
    wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh && \
    bash Anaconda3-2020.02-Linux-x86_64.sh -b && \
    rm Anaconda3-2020.02-Linux-x86_64.sh

# Path setting
ENV PATH $PATH:/root/anaconda3/bin

WORKDIR /root

# Install python library
ADD requirements.txt /root
RUN pip install -r requirements.txt

# Set root directory
WORKDIR ../
```

