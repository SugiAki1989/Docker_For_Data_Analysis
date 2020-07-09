# Chapter05 Docker For R

### はじめに

ここでは、R言語をDockerファイルからイメージビルドし、コンテナで起動する方法をまとめておきます。また、実際はDockerファイル作らなくても、RockerプロジェクトというものがRのコミュニティには存在し、そこにあるイメージを使えば、Rstudio Serverの環境構築から必要なパッケージのインストールまで簡単に行うことができます。今回はその中でも`rocker/tidyverse`イメージを紹介します。

### CentOSでRstudio Serverを動かす

ここではCentOS6でRstudio Serverを動かす方法をまとめておきます。ちなみにCentOS7,8で試してみたものの、うまく環境構築できませんでした…なので、CentOS6でR version 3.5.2の環境構築を行います。[こちらの方の内容](https://rf00.hatenablog.com/entry/2018/12/31/200709)を参考にさせていただきました。

まずはディレクトリとDockerファイルを作成します。

```text
→ cd ~/Desktop
→ mkdir R
→ cd R
→ touch Dockerfile
```

それではDockerファイルを書いていきます。必要なコマンドをインストールし、Rをインストールしてから、Rstudio Serverをダウンロードしてからインストールします。そして、ユーザーを作成し、`8787`ポートを開放します。

```text
# Base OS
FROM centos:6

# Print standard error output
RUN set -x

# Install utility command
RUN yum -y install epel-release && \
    yum -y install wget && \
    yum -y install sudo

# Rpm file from here
# https://rstudio.com/products/rstudio/download-server/redhat-centos/
RUN sudo yum -y install R && \
    wget -q https://download2.rstudio.org/rstudio-server-rhel-1.1.463-x86_64.rpm && \
    sudo yum -y install rstudio-server-rhel-1.1.463-x86_64.rpm

# Install R Packages 
RUN sudo -i R -e 'install.packages("dplyr", repos = "https://cran.ism.ac.jp/")'

# Create user(rstudio-user) and Set password(pass)
RUN adduser rstudio-user && \
    echo "rstudio-user:pass" | chpasswd

# Publish port
EXPOSE 8787
```

それではイメージビルドしていきます。10分くらいかかりますが、完了すると、下記のようにイメージが作成されています。

```text
➜ docker build -t rstudio-server .
【略】
Successfully built 97f4bcb3990b
Successfully tagged rstudio-server:latest


➜ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
rstudio-server      latest              97f4bcb3990b        43 seconds ago      1.86GB
centos              6                   d0957ffdf8a2        16 months ago       194MB
```

このイメージを使って、コンテナを起動して、`sudo rstudio-server verify-installation`コマンドでRstudio Serverを使えるようにします。

```text
➜ docker run -it --name rstudio-server -p 8787:8787 rstudio-server /bin/bash
[root@a984a979af02 /]# sudo rstudio-server verify-installation
Starting rstudio-server:                                   [  OK  ]
```

それでは`8787`ポートにローカルホストからアクセスします。さきほど設定したユーザーとパスワードを使ってログインします。

![](.gitbook/assets/rstudio-server-.png)

Dockerファイル内でインストールしたdplyrパッケージも、問題なくインストールされているのかを確認するために`library(dplyr)`を実行します。問題なく実行できています。

```text
> library("dplyr")

Attaching package: ‘dplyr’

The following objects are masked from ‘package:stats’:

    filter, lag

The following objects are masked from ‘package:base’:

    intersect, setdiff, setequal, union
```

次は、`rocker/tidyverse`イメージを使ってRstudio Serverの環境構築をおこないます。

### `rocker/tidyverse`イメージ





