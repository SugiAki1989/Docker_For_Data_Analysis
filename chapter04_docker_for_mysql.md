# Chapter04 Docker For MySQL

### はじめに

ここではMySQLをコンテナで起動する方法をまとめておきます。SQLの練習だったろ、バージョンによる違いなどを検証する際にはDockerでMySQLのコンテナを作ることは非常に便利です。

### MySQLコンテナの構築

MySQLコンテナを構築するには下記のコマンドを実行すれば、MySQLコンテナを起動できます。[MySQLのイメージ](https://hub.docker.com/_/mysql)はDocker Hubから取得します。`latest`なので、MySQL8のイメージになります。ホスト側のポートの設定は、13306にしています。これは私のホストの実行環境で、MySQLを3306で使用しているためです。`MYSQL_ROOT_PASSWORD`はrootユーザーのパスワードです。

```text
→ docker run -e MYSQL_ROOT_PASSWORD=pass -p 13306:3306 -d mysql:latest

Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql
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
Status: Downloaded newer image for mysql:latest
1c241beed989cc748631212e012743a217af1eb7495d2a1f9f6b61a22f59e351
```

MySQLのイメージからコンテナを構築する際には下記の環境変数を使用することが可能です。

| 環境変数名 | 内容 |
| :--- | :--- |
| MYSQL\_ROOT\_PASSWORD | 必須。MySQLのrootユーザーに設定されるパスワードを指定します。 |
| MYSQL\_DATABASE | オプション。イメージの起動時に作成されるデータベースの名前を指定できます。ユーザー/パスワードが指定されている場合、そのユーザーには、このデータベースへのGRANT ALL権限が付与されます。 |
| MYSQL\_USER, MYSQL\_PASSWORD | オプション。新しいユーザーを作成し、そのユーザーのパスワードを設定します。このユーザーには、MYSQL\_DATABASE変数で指定されたデータベースに対するroot権限が付与されます。ユーザーを作成するには、両方の変数が必要です。 |
| MYSQL\_ALLOW\_EMPTY\_PASSWORD | オプション。yes のような空でない値を設定すると、root ユーザの空白のパスワードでコンテナを起動することができます。 |
| MYSQL\_RANDOM\_ROOT\_PASSWORD | オプション。yes のような空でない値を設定すると、ルートユーザのためのランダムな初期パスワードを生成されます。  |
| MYSQL\_ONETIME\_PASSWORD | rootユーザーをinitを完了した時点で期限切れになるように設定し、初回ログイン時にパスワードの変更を強制します。空でない値を指定すると、この設定が有効になります。 |

問題なく起動しているようなので、コンテナに入ってみます。

```text
➜ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                NAMES
1c241beed989        mysql:latest        "docker-entrypoint.s…"   About a minute ago   Up About a minute   33060/tcp, 0.0.0.0:13306->3306/tcp   silly_wu
```

rootユーザーのパスワードを入力してMySQLにアクセスします。これでいつもどおりMySQLを利用できます。

```text
➜ docker exec -it 1c241beed989 bash 
root@1c241beed989:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

### MySQLコンテナのデータ

MySQLコンテナのデータは、コンテナを削除すると、もう一度コンテンを構築しても保存されていません。これを確認するために適当なデータベースとテーブルを作成し、値をインサートします。

```text
mysql> CREATE DATABASE sample;
Query OK, 1 row affected (0.01 sec)

mysql> USE sample;
Database changed

mysql> CREATE TABLE sample(id int, name varchar(255));

mysql> INSERT INTO sample values(1, 'Tanaka'), (2, 'Sato'), (3, 'Suzuki');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM sample;
+------+--------+
| id   | name   |
+------+--------+
|    1 | Tanaka |
|    2 | Suzuki |
|    3 | Sato   |
+------+--------+
3 rows in set (0.00 sec)
```

一度、コンテナから抜けてコンテナに再度入ってみます。この状態では、コンテナを削除していないので、先程作成したデータベースとテーブルは残っています。

```text
mysql> exit
Bye
root@1c241beed989:/# exit
exit

➜ docker exec -it 1c241beed989 bash 
root@1c241beed989:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sample             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

それではコンテナを削除して、再度、コンテナを起動します。

```text
➜ docker rm -f 1c241beed989
1c241beed989

➜ docker run -e MYSQL_ROOT_PASSWORD=pass -p 13306:3306 -d mysql:latest
334434eafd7b9b5ceea5b0cc96917266d9b12bf81d2cf207cb8048f160ab8b3d

➜ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                NAMES
334434eafd7b        mysql:latest        "docker-entrypoint.s…"   27 seconds ago      Up 26 seconds       33060/tcp, 0.0.0.0:13306->3306/tcp   reverent_greider
```

それでは新しいコンテナでMySQLにアクセスします。そうすると先程作成したデータベースとテーブルがなくなっておることが確認できます。

```text
➜ docker exec -it 334434eafd7b bash 
root@334434eafd7b:/# mysql -u root -p
Enter password: 

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```

このようにコンテナを作り直すとデータは残りません。これを回避するには、ボリュームをマウントする必要があります。それを試す前に不要なコンテナを削除しておきます。

```text
mysql> exit
Bye
root@334434eafd7b:/# exit
exit

➜ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                NAMES
334434eafd7b        mysql:latest        "docker-entrypoint.s…"   3 hours ago         Up 3 hours          33060/tcp, 0.0.0.0:13306->3306/tcp   reverent_greider

➜ docker rm -f 334434eafd7b
334434eafd7b
```

ホスト側のデスクトップに`mysql_data`というディレクトリを作成します。ここをコンテナのMySQLのマウント先に設定します。

```text
➜ mkdir mysql_data
 
➜ docker run -v "$PWD/mysql_data":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass -p 13306:3306 -d mysql:latest
d4937bb0fad70ff0907d7a24f8341fa3528255be8e310a309a1638a982eab62b
```

それではコンテナのMySQLにアクセスして、データベースとテーブルを作成して、

```text
➜ docker exec -it d4937bb0fad7 bash 
root@d4937bb0fad7:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE sample;
Query OK, 1 row affected (0.01 sec)

mysql> USE sample;
Database changed

mysql> CREATE TABLE sample(id int, name varchar(255));
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO sample values(1, 'New_Tanaka'), (2, 'New_Sato'), (3, 'New_Suzuki');
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM sample;
+------+------------+
| id   | name       |
+------+------------+
|    1 | New_Tanaka |
|    2 | New_Sato   |
|    3 | New_Suzuki |
+------+------------+
3 rows in set (0.00 sec)
```

コンテナを削除します。

```text
mysql> exit
Bye

root@d4937bb0fad7:/# exit
exit

➜ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                NAMES
d4937bb0fad7        mysql:latest        "docker-entrypoint.s…"   5 minutes ago       Up 5 minutes        33060/tcp, 0.0.0.0:13306->3306/tcp   sad_engelbart

➜ docker rm -f d4937bb0fad7
d4937bb0fad7
```

データベースとテーブルは作成して、デスクトップの`mysql_data`ディレクトリに保存しているので、これをコンテナを起動する際にマウントすれば先程の状態を再現できるはずです。中身はこのようになっています。

```text
➜ ls mysql_data/
#ib_16384_0.dblwr	binlog.000002		client-key.pem		ibtmp1			public_key.pem		undo_001
#ib_16384_1.dblwr	binlog.index		ib_buffer_pool		mysql			sample			undo_002
#innodb_temp		ca-key.pem		ib_logfile0		mysql.ibd		server-cert.pem
auto.cnf		ca.pem			ib_logfile1		performance_schema	server-key.pem
binlog.000001		client-cert.pem		ibdata1			private_key.pem		sys

```

それではマウント先を指定して、コンテナを起動し、先程のデータがあるか確認すると、データベースとテーブルも残っており、データを確認できます。

```text
➜ docker run -v "$PWD/mysql_data":/var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass -p 13306:3306 -d mysql:latest
43d476bea0f95c22986517ee39627bf1db30255ce22c6388ee509c5ca0cd5741

➜ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                NAMES
43d476bea0f9        mysql:latest        "docker-entrypoint.s…"   6 seconds ago       Up 5 seconds        33060/tcp, 0.0.0.0:13306->3306/tcp   dreamy_kilby

➜ docker exec -it 43d476bea0f9 bash 
root@43d476bea0f9:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE sample;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT * FROM sample;
+------+------------+
| id   | name       |
+------+------------+
|    1 | New_Tanaka |
|    2 | New_Sato   |
|    3 | New_Suzuki |
+------+------------+
3 rows in set (0.01 sec)
```

