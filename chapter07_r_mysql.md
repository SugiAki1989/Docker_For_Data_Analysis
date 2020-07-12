# Chapter07 Docker For R × MySQL

### はじめに

ここでは、RStudio ServerとMySQLのコンテナを起動し、Rとデータベースを接続した分析環境を構築します。RStudio Serverの環境構築には、`rocker/tidyverse`イメージを使用し、MySQLは`mysql:8.0.20`イメージを使用します。

### 全体図

ここでは、`data_env`というディレクトリを作成し、そこで作業を行います。最終的なディレクトリ構造は下記の通りです。`docker-compose.yml`に各コンテナ情報をまとめて記述し、`docker-compose up`でコンテナを複数起動する方法もありますが、わかりやすさのためにここでは`docker-compose.yml`を分けています。

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
│       ├── #ib_16384_0.dblwr　#コンテナ起動時に生成されるMySQLファイル
│       ├── #ib_16384_1.dblwr　#コンテナ起動時に生成されるMySQLファイル
【略】
└── R
    ├── Dockerfile
    ├── docker-compose.yml
    └── working_dir
```

MySQLにはコンテナ起動時にcsvのデータがインサートされるようにしています。これはチャプター04のMySQLのコンテナ環境の構築で説明した内容どおりです。RStudio Serverのコンテナ環境についても、チャプター05のRのコンテナ環境の構築で説明した内容どおりです。では、各コンテナの内容を確認しておきます。

### MySQLのコンテナ環境

再度、MySQLの環境について内容を確認しておきます。`docker-compose.yml`の中身はこのようになっています。`var_lib_mysql`ディレクトリでデータを永続化し、`init`ディレクトリで初回起動時にデータがインサートされるようにしています。ポートが`13306`なのは私のホスト環境にMySQLサーバーがあるので、ポートが衝突しないためです。

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

### Rのコンテナ環境

再度、Rの環境について内容を確認しておきます。`docker-compose.yml`の中身はこのようになっています。コンテナを起動した際に、`RMySQL`や`DBI`などのパッケージをインストールしたいので、`build: .`として、Dockerfileからビルドするようにしています。あとは、ワーキングディレクトリを作成しています。

```text
➜ cat docker-compose.yml 
version: '3'
services:
  r-studio-server:
    build: .
    restart: always
    ports:
      - "8787:8787"
    volumes:
      - ~/Desktop/data_env/R/working_dir:/home/rstudio/R_mounted_dir
```

Dockerfileの中身はこのようになっています。

```text
➜ cat Dockerfile 
FROM rocker/verse:latest

RUN R -e "install.packages(c('DBI', 'RMySQL'), dependencies=TRUE,  repos='https://cran.ism.ac.jp/')"
```

### コンテナの起動

それでは準備が整ったので、コンテナを起動していきます。まずはMySQLのコンテナを起動します。

```text
➜ cd data_env/MySQL/
➜ docker-compose up -d
Pulling mysql (mysql:8.0.20)...
8.0.20: Pulling from library/mysql
8559a31e96f4: Pull complete
d51ce1c2e575: Pull complete
c2344adc4858: Pull complete
fcf3ceff18fc: Pull complete
16da0c38dc5b: Pull complete
b905d1797e97: Pull complete
4b50d1c6b05c: Pull complete
c75914a65ca2: Pull complete
1ae8042bdd09: Pull complete
453ac13c00a3: Pull complete
9e680cd72f08: Pull complete
a6b5dc864b6c: Pull complete
Digest: sha256:8b7b328a7ff6de46ef96bcf83af048cb00a1c86282bfca0cb119c84568b4caf6
Status: Downloaded newer image for mysql:8.0.20
Creating mysql_mysql_1 ... done
```

コンテナが起動していることが確認できます。

```text
➜ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                NAMES
ab3ca9cd20b8        mysql:8.0.20        "docker-entrypoint.s…"   55 seconds ago      Up 54 seconds       33060/tcp, 0.0.0.0:13306->3306/tcp   mysql_mysql_1
```

データが入っているか確認しておきます。

```text
➜ docker exec -it ab3ca9cd20b8  bash
root@ab3ca9cd20b8:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select * from test_db.test_tl01;
ERROR 1146 (42S02): Table 'test_db.test_tl01' doesn't exist
mysql> select * from test_db.test_tbl1;
+------+-----------+
| code | name      |
+------+-----------+
| 001  | Tanaka    |
| 002  | Sato      |
| 003  | Suzuki    |
| 004  | Takahashi |
+------+-----------+
4 rows in set (0.02 sec)

mysql> select * from test_db.test_tbl2;
+------+-----+
| code | age |
+------+-----+
| 001  |  10 |
| 002  |  20 |
| 003  |  30 |
| 004  |  40 |
+------+-----+
4 rows in set (0.01 sec)

mysql> exit
Bye
root@ab3ca9cd20b8:/# exit
exit

```

問題なくデータもはいっているので、次はRStudio Serverのコンテナを起動します。

```text
➜ cd data_env/R/
➜ docker-compose up -d

➜ docker-compose up -d
Building r-studio-server
Step 1/2 : FROM rocker/verse:latest
latest: Pulling from rocker/verse
a4a2a29f9ba4: Pull complete
127c9761dcba: Pull complete
d13bf203e905: Pull complete
4039240d2e0b: Pull complete
febcccaaf348: Pull complete
00e7e5e4c4eb: Pull complete
93cbc5d9bf1e: Pull complete
6779fb005e80: Pull complete
86d76ba07063: Pull complete
6a92e6e10218: Pull complete
Digest: sha256:65cd0788d5882eed88bbe5a04049d3123cdec48816b05685ea2911c2c64f0b49
Status: Downloaded newer image for rocker/verse:latest
 ---> 992d935ef30d
Step 2/2 : RUN R -e "install.packages(c('DBI', 'RMySQL'), dependencies=TRUE,  repos='https://cran.ism.ac.jp/')"
 ---> Running in cedb48a15e56

R version 4.0.2 (2020-06-22) -- "Taking Off Again"
Copyright (C) 2020 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

> install.packages(c('DBI', 'RMySQL'), dependencies=TRUE,  repos='https://cran.ism.ac.jp/')
Installing packages into ‘/usr/local/lib/R/site-library’
(as ‘lib’ is unspecified)
trying URL 'https://cran.ism.ac.jp/src/contrib/DBI_1.1.0.tar.gz'
Content type 'application/x-gzip' length 633079 bytes (618 KB)
==================================================
downloaded 618 KB

trying URL 'https://cran.ism.ac.jp/src/contrib/RMySQL_0.10.20.tar.gz'
Content type 'application/x-gzip' length 52900 bytes (51 KB)
==================================================
downloaded 51 KB

* installing *source* package ‘DBI’ ...
** package ‘DBI’ successfully unpacked and MD5 sums checked
** using staged installation
** R
** inst
** byte-compile and prepare package for lazy loading
** help
*** installing help indices
** building package indices
** installing vignettes
** testing if installed package can be loaded from temporary location
** testing if installed package can be loaded from final location
** testing if installed package keeps a record of temporary installation path
* DONE (DBI)
* installing *source* package ‘RMySQL’ ...
** package ‘RMySQL’ successfully unpacked and MD5 sums checked
** using staged installation
Found mysql_config cflags and libs!
Using PKG_CFLAGS=-I/usr/include/mysql 
Using PKG_LIBS=-L/usr/lib/x86_64-linux-gnu -lmysqlclient -lpthread -lz -lm -lrt -lssl -lcrypto -ldl
** libs
rm -f RMySQL.so RMySQL-init.o connection.o db-apply.o driver.o exception.o fields.o result.o utils.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c RMySQL-init.c -o RMySQL-init.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c connection.c -o connection.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c db-apply.c -o db-apply.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c driver.c -o driver.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c exception.c -o exception.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c fields.c -o fields.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c result.c -o result.o
gcc -I"/usr/local/lib/R/include" -DNDEBUG -I/usr/include/mysql   -I/usr/local/include   -fpic  -g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g  -c utils.c -o utils.o
gcc -shared -L/usr/local/lib/R/lib -L/usr/local/lib -o RMySQL.so RMySQL-init.o connection.o db-apply.o driver.o exception.o fields.o result.o utils.o -L/usr/lib/x86_64-linux-gnu -lmysqlclient -lpthread -lz -lm -lrt -lssl -lcrypto -ldl -L/usr/local/lib/R/lib -lR
installing to /usr/local/lib/R/site-library/00LOCK-RMySQL/00new/RMySQL/libs
** R
** byte-compile and prepare package for lazy loading
** help
*** installing help indices
** building package indices
** testing if installed package can be loaded from temporary location
** checking absolute paths in shared objects and dynamic libraries
** testing if installed package can be loaded from final location
** testing if installed package keeps a record of temporary installation path
* DONE (RMySQL)

The downloaded source packages are in
	‘/tmp/Rtmpf0VQZF/downloaded_packages’
> 
> 
Removing intermediate container cedb48a15e56
 ---> d46fbad0d21f
Successfully built d46fbad0d21f
Successfully tagged r_r-studio-server:latest
WARNING: Image for service r-studio-server was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating r_r-studio-server_1 ... done
```

コンテナが起動していることが確認できます。

```text
➜ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                NAMES
851c8f7948d9        r_r-studio-server   "/init"                  About a minute ago   Up About a minute   0.0.0.0:8787->8787/tcp               r_r-studio-server_1
ab3ca9cd20b8        mysql:8.0.20        "docker-entrypoint.s…"   15 minutes ago       Up 15 minutes       33060/tcp, 0.0.0.0:13306->3306/tcp   mysql_mysql_1
```

それではRStudio Server\([http://localhost:8787/](http://localhost:8787/)\)にアクセスします。IDとパスワードは`rstudio`です。下記のコマンドを実行してMySQLからデータを取得します。

```text
library(RMySQL)
library(tidyverse)

con <- dbConnect(
  drv = RMySQL::MySQL(),
  dbname = "test_db",
  user = "root",
  password = "Pass",
  # docker network inspect mysql_default | jq '.[0].IPAM.Config[0].Gateway'の結果
  host = "172.19.0.1",
  port = 13306
)

dbGetQuery(con, "SELECT * FROM test_db.test_tbl1;")
#   code      name
# 1  001    Tanaka
# 2  002      Sato
# 3  003    Suzuki
# 4  004 Takahashi

dbGetQuery(con, "SELECT * FROM test_db.test_tbl2;")
#   code age
# 1  001  10
# 2  002  20
# 3  003  30
# 4  004  40
```

![](.gitbook/assets/sukurnshotto-2020-07-12-30644png%20%281%29.png)

ここで注意する必要があるのが`host`の設定です。MySQLのコンテナはローカルホストで起動していますが、ローカルホストの設定をするとコネクションを構築できません。本来であればこれらのコンテナのネットワークを構築するべきかもしれませんが、ここではアドレスを直接Rスクリプトで指定する方法にしています。

### Dockerネットワーク

1つのコンテナでは1つの機能を動かす構成にすると、コンテナ間の連携が必要になる場合があります。今回のように、MySQLとRStudio Serverの各コンテナを起動させ、MySQLのデータをRStudio Serverで取得するときなどです。コンテナ間通信を行う方法は、2つあります。

* Dockerネットワークを作成し、コンテナ名で接続できるようにする
* `--link`オプションを使用する\(レガシー機能なので非推奨\)

デフォルトで`bridge`というDockerネットワークがデフォルトで作成されています。`bridge`は、DNS設定がされていないため名前解決ができず、コンテナ名を使用したコンテナ間通信ができません。その場合、コンテナ起動時に自動で割り当てられるプライベートIPアドレスを用いる方法で通信します。新規でDockerネットワークを作成すると、コンテナ名を使用してコンテナ間で通信ができます。`docker network create <network name>`でネットワークを作成できます。

現状ではデフォルトのネットワークである`bridge`の中にMySQLもRStudio Serverも含まれています。

```text
➜ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1a76981eca72        bridge              bridge              local
e963e0cf6ead        host                host                local
7ca5740cb687        mysql_default       bridge              local
c1fc877b851e        r_default           bridge              local
```

RからMySQLにコネクションを構築するので、MySQLのネットワークを調べます。Gatewayにあるアドレスをコネクション構築時に使用する必要があります。

```text
➜ docker network inspect 7ca5740cb687
[
    {
        "Name": "mysql_default",
        "Id": "7ca5740cb6874a1cab4a2eba4071af4e2d7d86fd008f69ad22802bd8323bb8c5",
        "Created": "2020-07-11T15:54:21.3193136Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"　#これ
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "ab3ca9cd20b8f618c9913c1732e86989fdaf9462f232510864605705b9ed74b5": {
                "Name": "mysql_mysql_1",
                "EndpointID": "5f1601992f3058da4ed6ae1ee6753819676b8873e3a05cb6e7733304e645624e",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "mysql",
            "com.docker.compose.version": "1.25.5"
        }
    }
]
```

Gatewayの値を取得したければ、下記のようにすれば取得できます。

```text
➜ docker network inspect mysql_default | jq '.[0].IPAM.Config[0].Gateway'
"172.19.0.1"
```

このように複数のコンテナを起動して、コンテナ間通信する場合にはネットワークの設定も必要なるので注意が必要ですね。

### `docker-compose up`でコンテナを複数起動

`docker-compose`でコンテナを複数起動する方法に変更する方法をまとめておきます。さきほどのディレクトリ`data_env`ディレクトリをコピーして`data_env2`ディレクトリを作成します。ディレクトリ構成は下記のとおりです。

```text
➜ tree .
.
├── Dockerfile.rstudio
├── docker-compose.yml
├── init
│   ├── 10_ddl.sql
│   ├── 20_data_load.sh
│   ├── data1.csv
│   └── data2.csv
├── var_lib_mysql
└── working_dir

3 directories, 6 files
```

まずは各コンテナを通信するためのネットワーク`data_analysis_network`を作成します。

```text
➜ docker network create data_analysis_network
d3af08f4815e2bd5f9ad52395dafbc6158acef2d0979f43fe5ec0c0046ba8784

➜ docker network ls
NETWORK ID          NAME                    DRIVER              SCOPE
1a76981eca72        bridge                  bridge              local
d3af08f4815e        data_analysis_network   bridge              local
e963e0cf6ead        host                    host                local
7ca5740cb687        mysql_default           bridge              local
e6057ac87f12        none                    null                local
a77d133853e6        python_default          bridge              local
c1fc877b851e        r_default               bridge              local
```

コネクションを作成する際のアドレスを確認しておきます。

```text
➜ docker network inspect data_analysis_network | jq '.[0].IPAM.Config[0].Gateway'
"172.22.0.1"
```

それでは`docker-compose.yml`を下記のように書き換えます。注意する点としては、各サービスの項目に`networks: - data_analysis_network`が追加されている点と、コンテナの通信を利用するための`networks`項目を最下部に追記した点です。

```text
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
    networks:
      - data_analysis_network
    volumes:
      - ~/Desktop/data_env2/var_lib_mysql:/var/lib/mysql
      - ~/Desktop/data_env2/init:/docker-entrypoint-initdb.d

  r-studio-server:
    build:
      context: .
      dockerfile: Dockerfile.rstudio
    restart: always
    ports:
      - "8787:8787"
    networks:
      - data_analysis_network
    volumes:
      - ~/Desktop/data_env2/working_dir:/home/rstudio/R_mounted_dir

networks:
  data_analysis_network:
    external: true
```

これで準備が整ったので、コンテナを起動します。

```text
➜ docker-compose up -d
【略】
Creating data_env2_r-studio-server_1 ... done
Creating data_env2_mysql_1           ... done
```

コンテナの状況を確認します。

```text
➜ docker-compose ps
           Name                         Command             State                 Ports               
------------------------------------------------------------------------------------------------------
data_env2_mysql_1             docker-entrypoint.sh mysqld   Up      0.0.0.0:13306->3306/tcp, 33060/tcp
data_env2_r-studio-server_1   /init                         Up      0.0.0.0:8787->8787/tcp           
```

コンテナがバックグラウンドで起動していることが確認できたので、[http://localhost:8787/](http://localhost:8787/)にアクセスします。`data_analysis_network`のアドレスを使ってコネクションを作成します。

```text
library(RMySQL)
library(tidyverse)

con <- dbConnect(
  drv = RMySQL::MySQL(),
  dbname = "test_db",
  user = "root",
  password = "Pass",
  # docker network inspect data_analysis_network | jq '.[0].IPAM.Config[0].Gateway'の結果
  host = "172.22.0.1",
  port = 13306
)

dbGetQuery(con, "SELECT * FROM test_db.test_tbl1;")
#   code      name
# 1  001    Tanaka
# 2  002      Sato
# 3  003    Suzuki
# 4  004 Takahashi

dbGetQuery(con, "SELECT * FROM test_db.test_tbl2;")
#   code age
# 1  001  10
# 2  002  20
# 3  003  30
# 4  004  40
```

コンテナ間の通信もできていることが確認できました。使用後はコンテナを停止しておきます。

```text
➜ docker-compose stop
Stopping data_env2_mysql_1           ... done
Stopping data_env2_r-studio-server_1 ... done

➜ docker-compose ps
           Name                         Command             State    Ports
--------------------------------------------------------------------------
data_env2_mysql_1             docker-entrypoint.sh mysqld   Exit 0        
data_env2_r-studio-server_1   /init                         Exit 0   
```

