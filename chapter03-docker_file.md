# Chapter03 Docker File

### はじめに

ここではDockerファイルの基本的な内容と記述方法をまとめます。Dockerでは、インフラ構成＝コンテナの構成をDockerインストラクションに従って、Dockerファイルに記述することで、イメージを生成し、コンテナを構築します。Dockerファイルの[ベストプラクティス](http://docs.docker.jp/engine/userguide/eng-image/dockerfile_best-practice.html)はこちら。

### Dockerファイル

Dockerファイルは下記のDockerインストラクションによって記述されます。Dockerファイルはインフラ構成を記述するもので、ベースになるOSのイメージ、コンテナ内のコマンド、環境変数の設定などをDockerインストラクションなどを記述します。

| インストラクション | 説明 |
| :--- | :--- |
| FROM | ベースイメージの指定 |
| MAINTAINER | Dockerファイルのメンテナ |
| RUN | コマンド実行 |
| CMD | コンテナの実行コマンド |
| LABEL | ラベルの設定 |
| EXPOSE | ポートのエクスポート |
| ENV | 環境変数の設定 |
| ADD | ファイル/ディレクトリの追加。圧縮tarなどを解凍して追加したい場合。 |
| COPY | ファイルのコピー |
| VOLUME | ボリュームのマウント |
| ENTRYPOINT | コンテナの実行コマンド |
| USER | ユーザの指定 |
| WORKDIR | 作業ディレクトリの |
| ONBUILD | ビルド完了後に実行されるコマンド |

### Dockerファイルの作成

それではDockerファイルを作成していきましょう。今回はデスクトップに`docker_context`というディレクトリを作成し、その中に`Dockerfile`を作成します。

```text
➜ cd ~/Desktop/

➜ mkdir docker_context

➜ cd docker_context/

➜ touch Dockerfile
```

ベースとなるOSを記述する`FROM`は必須で先頭に記述します。今回はこれだけでイメージビルドして、Dockerイメージが作成されるか確認します。

```text
➜ cat Dockerfile 
FROM ubuntu:latest
```

`build`コマンドで`Dockerfile`があるディレクトリを指定します。この`Dockerfile`を格納している環境をビルドコンテキストなどと呼ばれます。

```text
➜ docker build -t build_ubuntu:0.0 ~/Desktop/docker_context/
Sending build context to Docker daemon  2.048kB
Step 1/1 : FROM ubuntu:latest
latest: Pulling from library/ubuntu
a4a2a29f9ba4: Pull complete 
127c9761dcba: Pull complete 
d13bf203e905: Pull complete 
4039240d2e0b: Pull complete 
Digest: sha256:35c4a2c15539c6c1e4e5fa4e554dac323ad0107d8eb5c582d6ff386b383b7dce
Status: Downloaded newer image for ubuntu:latest
 ---> 74435f89ab78
Successfully built 74435f89ab78
Successfully tagged build_ubuntu:0.0
```

イメージが作成されているか確認すると、たしかに`-t`で指定した`build_ubuntu:0.0`というイメージが作成されている事がわかります。

```text
➜ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
build_ubuntu        0.0                 74435f89ab78        2 weeks ago         73.9MB
ubuntu              latest              74435f89ab78        2 weeks ago         73.9MB
```

### Dockerイメージのレイヤー

さきほどubuntuのイメージを取得した際に、下記のように表示されたと思います。これは、ubuntuのイメージが4つのイメージレイヤーから構成されていることを意味します。

```text
a4a2a29f9ba4: Pull complete 
127c9761dcba: Pull complete 
d13bf203e905: Pull complete 
4039240d2e0b: Pull complete 
```

Dockerファイルからイメージを構成する場合、特定のDockerインストラクションごとにイメージレイヤーを構築しながらイメージを構成します。例えば`RUN`コマンドはイメージレイヤーを追加するコマンドです。下記のようにDockerイメージを書き換えて、

```text
➜ cat Dockerfile 
FROM ubuntu:latest

RUN apt-get update

RUN apt-get install -y nginx
```

イメージビルドしてみるとわかります。`-y`はインストール中の選択肢で`yes`を選択するオプションです。`Running in`の部分でイメージレイヤーが構成されています。

```text
~/Desktop/docker_context 
➜ docker build -t build_ubuntu:1.0 ~/Desktop/docker_context/
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu:latest
 ---> 74435f89ab78            # 1つ目のイメージレイヤ
Step 2/3 : RUN apt-get update
 ---> Running in 3de940847895 # 2つ目のイメージレイヤ
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [107 kB]
【略】
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [2900 B]
Fetched 14.1 MB in 6s (2317 kB/s)
Reading package lists...
Removing intermediate container 3de940847895
 ---> c4f8c8b3b51c
Step 3/3 : RUN apt-get install -y nginx
 ---> Running in 0c821298759c # 3つ目のイメージレイヤ
Reading package lists...
【略】
Processing triggers for libc-bin (2.31-0ubuntu9) ...
Removing intermediate container 0c821298759c
 ---> cc2bb24d43c0
Successfully built cc2bb24d43c0
Successfully tagged build_ubuntu:1.0

➜ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
build_ubuntu        1.0                 cc2bb24d43c0        About a minute ago   155MB
build_ubuntu        0.0                 74435f89ab78        2 weeks ago          73.9MB
ubuntu              latest              74435f89ab78        2 weeks ago          73.9MB
```

ここでDockerファイルを作成する際に便利な知識としてキャッシュというものがあります。これは、再度同じコマンドを実行するのであれば、キャッシュを使うことで2回目以降は処理が速くおわります。`curl`というパッケージを更に追加して、

```text
➜ cat Dockerfile 
FROM ubuntu:latest

RUN apt-get update

RUN apt-get install -y nginx

RUN apt-get install -y curl
```

イメージビルドしてみます。`Using cache`と表示され、先程インストールしていたものはキャッシュが利用されインストールがすぐに終了します。Dockerファイルを作成する際はこのキャッシュを使いながら作成することで効率化を図ることができます。

```text
➜ docker build -t build_ubuntu:2.0 ~/Desktop/docker_context/
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ubuntu:latest
 ---> 74435f89ab78
Step 2/4 : RUN apt-get update
 ---> Using cache
 ---> c4f8c8b3b51c
Step 3/4 : RUN apt-get install -y nginx
 ---> Using cache
 ---> cc2bb24d43c0
Step 4/4 : RUN apt-get install -y curl
 ---> Running in 8de1bda56284
Reading package lists...
【略】
Processing triggers for ca-certificates (20190110ubuntu1.1) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
Removing intermediate container 8de1bda56284
 ---> ccf5101224bc
Successfully built ccf5101224bc
Successfully tagged build_ubuntu:2.0
```

### Dockerインストラクション

冒頭に紹介したDockerインストラクションの各役割について、基本的なコマンドについて、もう少し詳しくまとめていきます。

#### RUNコマンド

まずは`RUN`コマンドからです。`RUN`コマンドはDockerイメージを生成する際に実行したいコマンドがある場合に記述します。

下記のように記述すれば、Dockerイメージを生成する際に、`apt-get update`、`apt-get install nginx`、`apt-get install curl`が実行されます。

```text
RUN apt-get update

RUN apt-get install -y nginx

RUN apt-get install -y curl
```

このままだと、Dockerファイルを構築する際に3つのイメージレイヤがRUNごとに生成されるので、まとめて記述します。

```text
RUN apt-get update && apt-get install -y \
    nginx \
    curl
```

#### CMDコマンド

`CMD`コマンドはDockerイメージを生成したあとに、コンテナ内でコマンドを実行する場合に利用します。例えばubuntuのイメージのDockerファイルの最後には、コンテナに入るとbashが起動するように設定されています。

```text
CMD ["/bin/bash"]
```

#### ENVコマンド

`ENV`コマンドは環境変数を設定するコマンドです。`ENV`コマンドで環境変数を指定しておけばコンテナを起動した際に環境変数を設定した状態で起動します。環境変数の指定の仕方はキーバリュー型で下記のように記述できます。

```text
FROM ubuntu:latest

ENV key1 "val1"
ENV key2=val2

CMD ["/bin/bash"]
```

イメージビルドしてコンテナを起動してみます。コンテナの環境変数にDockerファイルで指定した環境変数が指定されていることがわかります。

```text
➜ docker build -t build_ubuntu ~/Desktop/docker_context/
Sending build context to Docker daemon  2.048kB
【略】
Successfully built 6ce0d7130333
Successfully tagged build_ubuntu:latest

➜ docker run -it --rm build_ubuntu bash
root@fefd4c983663:/# env | grep "key"
key2=val2
key1=val1
```

#### WORKDIRコマンド

`WORKDIR`コマンドはコマンドを実行するディレクトリを指定するコマンドです。`WORKDIR`コマンドで作業ディレクトリを指定して、テキストファイルをDockerファイルから作ってみます。

```text
FROM ubuntu:latest

WORKDIR /sample

RUN touch sample.txt

CMD ["/bin/bash"]
```

イメージビルドしていきます。コンテナを起動すると`WORKDIR`コマンドで指定したディレクトリにアクセスすることになります。テキストファイルも問題なく生成されていることが確認できます。

```text
➜ docker build -t build_ubuntu ~/Desktop/docker_context/
【略】
Successfully built db587f73412b
Successfully tagged build_ubuntu:latest

➜ docker run -it --rm build_ubuntu bash
root@af616c59e1b5:/sample# ls  
sample.txt
```

