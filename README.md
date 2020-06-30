# Docker For Data Analysis

## はじめに

この本は、データ分析環境を構築するためにDockerの基礎的な部分からMySQL、R、Pythonでのデータ分析環境構築まで、個人的に学習した内容をまとめているものです。

備忘録みたいなものなので、私は特段Dockerや仮想化技術に強いわけでもなく、初学者の学習ノートなのでトンチンカンな記述もあるはずです。基本的には下記のサイトの内容をもとに、自分用にまとめています。

### 参考書籍とサイト

下記を参考に、Dockerの利用方法をまとめています。

* [Docker Docs](https://docs.docker.com/)
* [Docker Docs Ja](http://docs.docker.jp/index.html#)

### 免責事項

ここにまとめられているサンプルコードを実行したことによって、万一いかなる損害が発生したとしても、著者はいかなる責任も負いません。すべて自己責任でお使いいただけますと幸いです。

### 実行環境

macOSの環境で実行しています。バージョンは下記の通りです。

```text
➜ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.5
BuildVersion:	19F101
```

Dockerのバージョンは下記のとおりです。

```text
➜ docker version
Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b
 Built:             Wed Mar 11 01:21:11 2020
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.8
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.17
  Git commit:       afacb8b
  Built:            Wed Mar 11 01:29:16 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

[Docker](https://www.docker.com/)のサイトにアクセスして、Docker For Macのインストールは完了させておいてください。



