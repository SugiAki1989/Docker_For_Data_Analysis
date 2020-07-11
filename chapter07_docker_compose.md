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

再度、MySQLの環境について内容を確認しておきます。`docker-compose.yml`の中身はこのようになっています。`var_lib_mysql`ディレクトリでデータを永続化し、`init`ディレクトリで初回起動時にデータがインサートされるようにしています。

```text
➜ cat docker-compose.yml
version: '3'
services:
  mysql:
    image: mysql:8.0.20
    restart: always
    environment:
      MYSQL_DATABASE: test_db
      MYSQL_ROOT_PASSWORD: Pass
    ports:
      - "13306:3306"
    volumes:
      - ~/Desktop/data_env/MySQL/var_lib_mysql:/var/lib/mysql
      - ~/Desktop/data_env/MySQL/init:/docker-entrypoint-initdb.d
```

`10_ddl.sql`でテーブルを定義して、`20_data_load.sh`でデータをインポートしています。MySQL8を使うので、`set global local_infile = 1;`を忘れないようにします。

```text
➜ cat init/10_ddl.sql 
set global local_infile = 1;

create table if not exists test_tbl1(
  `code` char(3) not null,
  `name` varchar(80) not null,
  primary key(`code`)
) engine=innodb default charset=utf8;

create table if not exists test_tbl2(
  `code` char(3) not null,
  `age` int not null,
  primary key(`code`)
) engine=innodb default charset=utf8; 

-----------------

➜ cat init/20_data_load.sh 
mysql -uroot -pPass --local-infile=1 test_db -e "LOAD DATA LOCAL INFILE '/docker-entrypoint-initdb.d/data1.csv' INTO TABLE test_tbl1 FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\n' IGNORE 1 LINES"

mysql -uroot -pPass --local-infile=1 test_db -e "LOAD DATA LOCAL INFILE '/docker-entrypoint-initdb.d/data2.csv' INTO TABLE test_tbl2 FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\n' IGNORE 1 LINES"
```





