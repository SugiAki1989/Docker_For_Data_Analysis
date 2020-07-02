# Chapter03 Docker File

### はじめに

ここではDockerファイルの基本的な内容と記述方法をまとめます。Dockerでは、インフラ構成＝コンテナの構成をDockerインストラクションに従って、Dockerファイルに記述することで、イメージを生成し、コンテナを構築します。Dockerファイルの[ベストプラクティス](http://docs.docker.jp/engine/userguide/eng-image/dockerfile_best-practice.html)はこちら。

### Dockerインストラクション



| インストラクション | 説明 |
| :--- | :--- |
| FROM | ベースイメージの指定 |
| MAINTAINER | Dockerファイルのメンテナ |
| RUN | コマンド実行 |
| CMD | コンテナの実行コマンド |
| LABEL | ラベルの設定 |
| EXPOSE | ポートのエクスポート |
| ENV | 環境変数の設定 |
| ADD | ファイル/ディレクトリの追加 |
| COPY | ファイルのコピー |
| VOLUME | ボリュームのマウント |
| ENTRYPOINT | デーモン実行 |
| USER | ユーザの指定 |
| WORKDIR | 作業ディレクトリの指定 |
| ONBUILD | ビルド完了後に実行される命令 |

