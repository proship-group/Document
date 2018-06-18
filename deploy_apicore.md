# apicoreのデプロイ手順

## 概要

#### hexalink-k8sのファイル構成

```bash
~/go/src/hexalink-k8s/microservices/apicore
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
$ cd ~/go/src/apicore
```

### Configurationファイル作成

バックアップ取得

```
$ cp config.env config.env.bk.`date "+%Y%m%d_%I%M%S"`
# 「No such file or directory」が表示しても無視してください。

```
templateファイル`.example`をコピーし開く。

```bash
$ cp config.env.example config.env
$ vi config.env
```

下記の設定を行う：

```bash
# Emails template ID
INVITE_EMAIL=
SIGNUP_EMAIL=
FORGOT_EMAIL=
INQUIRY_EMAIL=
```

各template IDは[Sendgrid Transactional Templates](https://sendgrid.com/templates)で確認してください。

#### Service Account Key

#####バックアップ取得

```
$ cp gcp/secrets/gcp-service-account-${ENVIRONMENT}.json gcp/secrets/gcp-service-account-${ENVIRONMENT}.json.bk.`date "+%Y%m%d_%I%M%S"`
# 「No such file or directory」が表示しても無視してください。
```

#####サービスアカウントキーの作成
[Elasticsearch](deploy_elasticsearch.md)のデプロイ手順で サービスアカウントキーがすでに取得したため、それを`gcp/secrets/`ディレクトリにコピーする。下記のコマンドを実行してください。

```bash
$ cp ~/go/src/hexalink-k8s/middlewares/elasticsearch/secrets/${ENVIRONMENT}/service-account.json gcp/secrets/gcp-service-account-${ENVIRONMENT}.json
```

> （翻訳保留中）If you need to change the filename of the `.json` service account key file, you need to also update the `env` variable filename in `K8s Deployment` for `apicore`. The `env` variable name is `GCP_KEYFILE_NAME`

### Imageビルド

下記のコマンドを実行し`apicore` microserviceのimageをビルドする。

```bash
$ go get -v ./...
$ ./do.sh build_push
```

このコマンドを実行することでdocker imageをビルドし、そのimageをプロジェクト ($PROJECT_ID)のGCRリポジトリにpushする。

## Deployment

deployments directoryに移動する

```bash
$ cd ~/go/src/hexalink-k8s/microservices/apicore
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
podの起動が少し時間がかかるので、3分ほどお待ちください

```bash
$ kubectl get pods -l component=microservice,role=apicore
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認

```