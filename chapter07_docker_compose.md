# Chapter07 Docker Compose

### はじめに

ここでは、RStudio ServerとMySQLのコンテナを起動し、Rとデータベースを接続した分析環境を構築します。RStudio Serverの環境構築には、`rocker/tidyverse`イメージを使用し、MySQLは`mysql:8.0.20`イメージを使用します。

### 全体図

ここでは、`data_env`というディレクトリを作成し、そこで作業を行います。最終的なディレクトリ構造は下記の通りです。

```text
➜ tree data_env
data_env
├── MySQL
│   ├── docker-compose.yml
│   ├── init
│   │   ├── 10_ddl.sql
│   │   ├── 20_data_load.sh
│   │   ├── data1.csv
│   │   └── data2.csv
│   └── var_lib_mysql
│       ├── #ib_16384_0.dblwr
│       ├── #ib_16384_1.dblwr
【略】
└── R
    ├── Dockerfile
    ├── docker-compose.yml
    └── working_dir
```

MySQLにはコンテナ起動時にcsvのデータがインサートされるようにしています。これはチャプター04のMySQLのコンテナ環境の構築で説明した内容どおりです。RStudio Serverのコンテナ環境についても、チャプター05のRのコンテナ環境の構築で説明した内容どおりです。では、各コンテナの内容を確認しておきます。

### MySQL





