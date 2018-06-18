# configctlのデプロイ手順

## 概要

#### ファイル構成

```bash
~/go/src/hexalink-k8s/microservices/configctl
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
$ cd ~/go/src/configctl
```

### Configurationファイル作成

バックアップ取得

```
$ cp config.env config.env.bk.`date "+%Y%m%d_%I%M%S"`
# 「No such file or directory」が表示しても無視してください。

$ cp apikv.yml apikv.yml.bk.`date "+%Y%m%d_%I%M%S"`
# 「No such file or directory」が表示しても無視してください。

$ cp database.yml database.yml.bk.`date "+%Y%m%d_%I%M%S"`
# 「No such file or directory」が表示しても無視してください。
```
templateファイル`.example`をコピーする

```bash
$ cp config.env.example config.env
$ cp apikv.yml.example apikv.yml
$ cp database.yml.example database.yml
```

 `.yml`ファイルを開き、デプロイ中の環境のアイテムを設定する。

```
# ファイルを開く
$ vi <ファイル名>
# 「i」を入力し編集モードに入る。編集完了すれば「esc」を押して閲覧モードに戻る。「:wq」を入力し保存して閉じる。編集を廃棄する場合、「:q!」を入力してください。
```
> `stg` 環境の場合、 `dev:*`のアイテムを設定する
>
> `prod` 環境の場合、`production:*`のアイテムを設定する

#### apikv.yml

**`sentry_dsn`** アイテムに関して、そのvalueは`web-ui` microserviceに使われる。そのvalueはエンドユーザに公開するため、**`Public DSN`**に設定してください。

apikv.ymlに下記の情報がなければ、追記してください。

```
local:messageBackend:
    eventgridHost: "ws://localhost:5001"
dev:messageBackend:
    eventgridHost: "wss://nextpit.ml"
production:messageBackend:
    eventgridHost: "wss://nextpit.tk"

local:externalIntegrations:
    useExtension: 'true'
    exUIHostURL: 'http://localhost:9007'
    exUIScriptPath: '/public/export.min.js'
    exUIRoutingPath: '/external_integration'
    exUIRequestPrefix: '/auditui'
    exAPIHostURL: 'http://localhost:9091'
    exAPIRoutingPath: '/auditcore/api/v1'
    exAPIRequestPrefix: '/api/v1'
dev:externalIntegrations:
    useExtension: 'false'
    exUIHostURL: 'http://auditui:9007'
    exUIScriptPath: '/public/export.min.js'
    exUIRoutingPath: '/external_integration'
    exUIRequestPrefix: '/auditui'
    exAPIHostURL: 'http://auditcore:9091'
    exAPIRoutingPath: '/auditcore/api/v1'
    exAPIRequestPrefix: '/api/v1'
production:externalIntegrations:
    useExtension: 'false'
    exUIHostURL: 'http://auditui:9007'
    exUIScriptPath: '/public/export.min.js'
    exUIRoutingPath: '/external_integration'
    exUIRequestPrefix: '/auditui'
    exAPIHostURL: 'http://auditcore:9091'
    exAPIRoutingPath: '/auditcore/api/v1'
    exAPIRequestPrefix: '/api/v1'
```


#### database.yml

必要な場合、**各middlewareのパスワードを忘れずに設定してください。** 

middlewareのデプロイ中でアイテムを変更しない限り、database.ymlのアイテムは下記のように設定してください。

>注意
>
>stringのバリューは必ずダブルコンテーション`""`で囲む。バリューと`:`の間に一つのスペースにあける。
>
>例：`username: "devops"`


```yaml
mongodb:
  dev:
    host: "mongos"
    port: 27017
    username: "devops"
    database: "beee-dev"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/mongodb/${ENVIRONMENT}_password.txt」
  production:
    host: "mongos"
    port: 27017
    username: "devops"
    database: "beee-dev"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/mongodb/${ENVIRONMENT}_password.txt」

neo4j:
  dev:
    host: "d-neo4j-0"
    port: 7474
    username: "neo4j"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/neo4j/${ENVIRONMENT}_password.txt」
  production:
    host: "d-neo4j-0"
    port: 7474
    username: "neo4j"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/neo4j/${ENVIRONMENT}_password.txt」

redis:
  dev:
    host: "a-redis"
    port: 6379
    user: 
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/redis/${ENVIRONMENT}_password.txt」
  production:
    host: "a-redis"
    port: 6379
    user:
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/redis/${ENVIRONMENT}_password.txt」

mysql:
  dev:
    host: "mysql-0.mysql"
    port: 3306
    username: "devops"
    database: "b-eee-dev"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/mysql/secrets/user.password」
  production:
    host: "mysql-0.mysql"
    port: 3306
    username: "devops"
    database: "b-eee-dev"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/mysql/secrets/user.password」

nsq:
  dev:
    host: "a-nsq-1:4150"
  production:
    host: "a-nsq-1:4150"

elasticsearch:
  dev:
    host: "http://elasticsearch:9200"
    username: "elastic"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/elasticsearch/${ENVIRONMENT}_password.txt」
  production:
    host: "http://elasticsearch:9200"
    username: "elastic"
    password: ←右記のコマンドを実行した結果を記入「$ cat $GOPATH/src/hexalink-k8s/middlewares/elasticsearch/${ENVIRONMENT}_password.txt」

eventgrid:
  local:
    host: "localhost:5001"
  dev:
    host: "beee-eventgrid:5001"
  production:
    host: "beee-eventgrid:5001"
```

> 参考用

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
$ cd ~/go/src/hexalink-k8s/microservices/configctl
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
podの起動が少し時間がかかるので、3分ほどお待ちください

```

$ kubectl get pods -l component=microservice,role=configctl
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認
```