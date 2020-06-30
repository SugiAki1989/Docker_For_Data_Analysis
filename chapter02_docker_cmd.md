# Chapter02 Docker Command

### はじめに

ここではDockerの基本的なコマンドの使い方に焦点をあてて、まとめていきます。Dockerでは、イメージの取得などをはじめ、コンテナの起動から停止、削除までDockerコマンドで行います。

### Docker Hub

いきなりDockerコマンドではないですが、Docker Hubについて簡単に触れておきます。これは、Docker社が運営しているDockerイメージのレポジトリーサービスです。Docker Hubがあるので、手元のローカルPC、クラウド環境、様々な環境にイメージを共有することが可能です。

例えばRのイメージであれば下記にアクセスすることで情報を確認できます。

* [https://hub.docker.com/\_/r-base?tab=description](https://hub.docker.com/_/r-base?tab=description)

Tagsのページを見ると何やら、異なるバージョンのRのイメージが確認できます。

![](.gitbook/assets/sukurnshotto-2020-06-30-164954png.png)

Dockerでは、イメージを管理する際にタグを利用します。例えば、タグを指定せずにDocker Hubからイメージを取得すると、1番上にある`latest`が取得されます。そのため、バージョンを指定してイメージを取得したい場合は、下記のようにタグでバージョンを指定します。

```text
➜ docker run r-base:4.0.2
```

### pullコマンド

それではDockerコマンドを見ていきましょう。まずは`pull`コマンドです。このコマンドはDoker HubからDockerイメージを取得する際に利用するコマンドです。DockerイメージをDoker Hubではなく他の場所で管理している場合にも利用できます。下記は[tensorflow.org](https://www.tensorflow.org/install/docker?hl=ja)で公開されている機械学習用のフレームワークであるtensorflowのDockerイメージを取得する際のコマンドです。

```text
→ docker pull tensorflow/tensorflow:latest-gpu-jupyter
```

オプションは下記の通り用意されています。`-a`をつけると全てのイメージを取得できるようです。

```text
➜ docker pull --help

Usage:	docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
  -q, --quiet                   Suppress verbose output
```

