# importerのデプロイ手順

## 概要

#### ファイル構成

```bash
~/go/src/hexalink-k8s/microservices/importer
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
$ cd ~/go/src/importer
```

###  Configurationファイル作成

templateファイル`.example`をコピーする

```bash
$ cp config.env.example config.env
```

デフォルトのvalueはそのままで良い。

### Imageビルド
下記のコマンドを実行し`importer ` microserviceのimageをビルドする。

```bash
$ go get -v ./...
$ ./do.sh build_push
```

このコマンドを実行することでdocker imageをビルドし、そのimageをプロジェクト ($PROJECT_ID)のGCRリポジトリにpushする。

## Deployment

deployments directoryに移動する

```bash
$ cd ~/go/src/k8s-deployments/microservices/importer
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
$ kubectl get pods -l component=microservice,role=importer
```