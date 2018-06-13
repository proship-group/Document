# mailfetcherのデプロイ手順

## 概要

#### ファイル構成

```bash
~/go/src/hexalink-k8s/microservices/mailfetcher
├── do.sh -> ../../do.sh
├── prod.yaml
└── stg.yaml
```

> 参考用

## セットアップ

作業を進める前に、環境変数の設定を確認してください。
[環境変数設定](prepare_envvars.md)を参考してください.
名前が`${PROJECT_ID}`の**GCS Bucket**は作成済みのことを確認してください。作成していない場合、[Create GCS Buckets](create_gcs_buckets.md)を参照してください。

appのdirectoryに移動 

```bash
$ cd ~/go/src/mailfetcher
```

### Configurationファイル作成

templateファイル`.example`をコピーし開く。

```bash
$ cp config.env.example config.env
```

各valueはデフォルトのままで良い

### Imageビルド

下記のコマンドを実行し`mailfetcher ` microserviceのimageをビルドする。


```bash
$ go get -v ./...
$ ./do.sh build_push
```

このコマンドを実行することでdocker imageをビルドし、そのimageをプロジェクト ($PROJECT_ID)のGCRリポジトリにpushする。

## Deployment

deployments directoryに移動する

```bash
$ cd ~/go/src/k8s-deployments/microservices/mailfetcher
```

### Node-Poolを選択

デフォルトでpodが`default-pool` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**

### K8s Deployment作成

下記のコマンドを実行してください。

```bash
$ ./do.sh deploy_microservice
```
このコマンドを実行するとデプロイ・リソースのYAMLファイルが表示される。デプロイ・リソースが正しいかどうかをそれで確認できる。

#### Pod確認

```bash
$ kubectl get pods -l component=microservice,role=mailfetcher
```