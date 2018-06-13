# adminToolのデプロイ手順

## 概要

#### ファイル構成

```bash
~/go/src/k8s-deployments/microservices/adminTool
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
$ cd ~/go/src/admintool
```

### Configurationファイル作成

全ての設定は`configctl`から取得するために、設定不要。


### Imageビルド

`adminTool`をビルドする前に、.NETのインストール状況の確認が必要。まだインストールしていない場合、[Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/linuxubuntu)を参照してください。インストールした後に、imageをビルドする。

下記のコマンドを実行し`adminTool ` microserviceのimageをビルドする。

```bash
$ dotnet restore
$ sudo npm -g install webpack
$ npm install
$ ./do.sh init
$ docker build --no-cache -t gcr.io/${PROJECT_ID}/beee-admintool:latest .
$ gcloud docker -- push gcr.io/${PROJECT_ID}/beee-admintool:latest
```

このコマンドを実行することでdocker imageをビルドし、そのimageをプロジェクト ($PROJECT_ID)のGCRリポジトリにpushする。

## Deployment

Go to deployments directory

```bash
$ cd ~/go/src/k8s-deployments/microservices/adminTool
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
$ kubectl get pods -l component=microservice,role=admintool
```