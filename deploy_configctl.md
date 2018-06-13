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

templateファイル`.example`をコピーする

```bash
$ cp config.env.example config.env
$ cp apikv.yml.example apikv.yml
$ cp database.yml.example database.yml
```

 `.yml`ファイルを開き、デプロイ中の環境のアイテムを設定する。

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
    eventgridHost: "wss://nextpit.tk"
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

middlewareのデプロイ中でアイテムを変更しない限り、database.ymlのアイテムは下記のように設定してください。（要編集）


```yaml
mongodb:
  dev:
    host: mongos
    port: 27017
    username: "devops"
    database: "beee-dev"
  production:
    host: mongos
    port: 27017
    username: "devops"
    database: "beee-dev"

neo4j:
  dev:
    host: d-neo4j-0
    port: 7474
    username: "neo4j"
  production:
    host: d-neo4j-0
    port: 7474
    username: "neo4j"

redis:
  dev:
    host: a-redis
    port: 6379
  production:
    host: a-redis
    port: 6379

mysql:
  dev:
    host: mysql-0.mysql
    port: 3306
    username: "devops"
    database: "b-eee-dev"
  production:
    host: mysql-0.mysql
    port: 3306
    username: "devops"
    database: "b-eee-dev"

nsq:
  dev:
    host: "a-nsq-1:4150"
  production:
    host: "a-nsq-1:4150"

elasticsearch:
  dev:
    host: "http://elasticsearch:9200"
    username: elastic
  production:
    host: "http://elasticsearch:9200"
    username: elastic

eventgrid:
  local:
    host: localhost:5001
  dev:
    host: beee-eventgrid:5001
  production:
    host: beee-eventgrid:5001
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

```bash
$ kubectl get pods -l component=microservice,role=configctl
```