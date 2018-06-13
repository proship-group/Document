# actionScriptのデプロイ手順

## 概要

#### ファイル構成

```bash
~/go/src/k8s-deployments/microservices/actionScript
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
$ cd ~/go/src/actionScript
```


### Imageビルド

下記のコマンドを実行し`actionScript ` microserviceのimageをビルドする。


```bash
$ docker build --no-cache -t gcr.io/${PROJECT_ID}/beee-actionscript:latest -f docker/app/Dockerfile .
$ gcloud docker -- push gcr.io/${PROJECT_ID}/beee-actionscript:latest
```

このコマンドを実行することでdocker imageをビルドし、そのimageをプロジェクト ($PROJECT_ID)のGCRリポジトリにpushする。

## Deployment

deployments directoryに移動する

```bash
$ cd ~/go/src/k8s-deployments/microservices/actionScript
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
$ kubectl get pods -l component=microservice,role=actionscript
```