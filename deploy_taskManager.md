# taskManagerのデプロイ手順

## 概要

#### ファイル構成

```bash
~/go/src/hexalink-k8s/microservices/taskManager
├── do.sh -> ../../do.sh
├── prod.yaml
└── stg.yaml
```

> 参考用

## セットアップ

作業を進める前に、環境変数の設定を確認してください。
[環境変数設定](prepare_envvars.md)を参考してください.

appのdirectoryに移動

```bash
$ cd ~/go/src/taskManager
```

### Configurationファイル作成

templateファイル`.example`をコピーし開く。

```bash
$ cp config.env.example config.env
$ vi config.env
```

下記のvalueを設定する

```bash
...
UNREAD_MESSAGE_TEMP=
```

[Sendgrid Transactional Templates](https://sendgrid.com/templates)を参照しtemplate IDを`UNREAD_MESSAGE_TEMP=`に設定する。

他のアイテムはデフォルト値で良い。

### Imageビルド

下記のコマンドを実行し`configctl` microserviceのimageをビルドする。

```bash
$ go get -v ./...
$ ./do.sh build_push
```

このコマンドを実行することでdocker imageをビルドし、そのimageをプロジェクト ($PROJECT_ID)のGCRリポジトリにpushする。

## Deployment

deployments directoryに移動する

```bash
$ cd ~/go/src/k8s-deployments/microservices/taskManager
```

### Node-Poolを選択

デフォルトでpodが`default-pool` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**

### K8s Deployment作成

下記のコマンドを実行する

```bash
$ ./do.sh deploy_microservice
```

このコマンドを実行するとデプロイ・リソースのYAMLファイルが表示される。デプロイ・リソースが正しいかどうかをそれで確認できる。

#### Pod確認

```bash
$ kubectl get pods -l component=microservice,role=taskmanager
```