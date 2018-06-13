# Elasticsearch

## 概要

こちらのファイルでElasticsearch Clusterを3つのMaster、2つのClientと2つのData Nodeで作成する。


#### ファイル構成

```bash
elasticsearch
.
├── README.md
├── config
│   ├── globalsearch_template.json
│   ├── synonym.txt
│   └── userdict_ja.txt
├── do.sh
├── es-client
│   ├── prod.yaml
│   └── stg.yaml
├── es-data-stateful.yaml
├── es-data-svc.yaml
├── es-discovery-svc.yaml
├── es-master.yaml
├── es-svc.yaml
└── secrets
    ├── prod
    │   └── service-account.json
    └── stg
        └── service-account.json
```

> 参考用


## セットアップ

deployments directoryに移動する

```bash
$ cd ~/go/src/hexalink-k8s/middlewares/elasticsearch
```

### ConfigMapsを作成

```bash
$ kubectl create configmap elasticsearch-config --from-file=config
```

### backupを有効化

backupを有効化させ、そのbackupをGCSにアップロードするのに、Elasticsearch用のService Accountの作成が必要。

> [サービス アカウント](https://console.cloud.google.com/projectselector/iam-admin/serviceaccounts)でサービスアカウントのキーを確認する。名前が`Compute Engine default service account`のサービスアカウントしかない場合、[GCPサービスアカウント作成](create_service_account.md) 手順を参考しサービスアカウントを作ってください。すでに`roles/storage.admin`のservice roleのサービスアカウントをお持ちの場合、新しいサービスアカウントを作成せずにそれを使っても構わない。

サービスアカウントキーの作成に関して、画面での操作方法とコマンドラインでの実行方法がある。



####画面での操作方法：

作成（新しいサービスアカウントを作成する場合）：（未確認）

ダウンロード（既存のサービスアカウントを使う場合）：
[サービス アカウント画面](https://console.cloud.google.com/projectselector/iam-admin/serviceaccounts)に該当のサービスアカウントを選択し、`鍵を作成`をクリック。「キーのタイプ」を「JSON」を選択して`作成`ボタンを押す。ダウンロードしたjsonファイルを`secrets/`にコピーし、ファイル名を`service-account.json`に変更する。

---

####コマンドラインでの実行方法：

下記のコマンドを実行

```bash
$ gcloud iam service-accounts keys create service-account.json --iam-account <メールアドレス>
#メールアドレスは「https://console.cloud.google.com/projectselector/iam-admin/serviceaccounts」で確認してください。

$ cp service-account.json secrets/${ENVIRONMENT}
$ rm service-account.json

```

___注意:サービスアカウントキーはセキュリティ問題につながるため、お取扱について十分に注意してください。___

### Secret作成

```bash
$ kubectl create secret generic es-secret --from-file=secrets/${ENVIRONMENT}
```

### Node-Pool選択

デフォルトでpodが`pool-db0` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**

### Deployment

```bash
$ kubectl create -f es-discovery-svc.yaml
$ kubectl create -f es-master.yaml

# deployment status確認
$ kubectl rollout status -f es-master.yaml

$ kubectl create -f es-svc.yaml
$ kubectl create -f es-client/${ENVIRONMENT}.yaml

# deployment status確認
$ kubectl rollout status -f es-client/${ENVIRONMENT}.yaml

$ kubectl create -f es-data-svc.yaml
$ kubectl create -f es-data-stateful.yaml
```

下記のコマンドでcomponentを確認する

```bash
# all components
$ kubectl get pods -l component=elasticsearch

# client components
$ kubectl get pods -l component=elasticsearch,role=client
# master components
$ kubectl get pods -l component=elasticsearch,role=master
# data components
$ kubectl get pods -l component=elasticsearch,role=data
```

## Post セットアップ

ユーザ`elastic` のデフォルトパスワードは`changeme`。
> 必要な場合、より安全なパスワードに変更してください。

### デフォルトパスワードを変更する

> 下記のコマンドは必ず「Deployment」手順を完了してから実行してください。

下記のコマンドは`es-client/${ENVIRONMENT}.yaml`での`redinessProbe`の`Authorization` ヘッダーを変更する。

```yaml
readinessProbe:
    httpGet:
    path: /_cluster/health
    port: http
    httpHeaders:
        - name: Authorization
        value: Basic ZWxhc3RpYzpjaGFuZ2VtZQ== #dont-remove-this-comment: elastic auth
    initialDelaySeconds: 40
    timeoutSeconds: 5
```

> セットアップにより `x-pack`を有効化させるため、こちらのヘッダーが必要になる。この認証を無効化するには、全てのyamlファイルにある`x-pack` を削除してください。
`.spec.container[0].env`->`ES_PLUGINS_INSTALL`

下記のコマンドで`Authorization`を新しいパスワードに変更する
`新しいパスワード`に何も入力しなければ、自動的にランダムのパスワードを作成する。

```bash
$ ./do.sh change_password [新しいパスワード]
```

これで${ENVIRONMENT}_password.txtフォーマットのファイルがカレントディレクトリに作成される。その中に新しいパスワードを含める。必要な場合、そのファイルを切り取りしてください。

下記のコマンドを実行しパスワードをアップデートする

```bash
$ ./do.sh update_password ${ENVIRONMENT}_password.txt
Closed port-forward (killed portforward.pid)
```

このコマンドはelasticsearch clientをlocalhostに`port-forward`し、curlコマンドを実行しelasticsearchのパスワードを変更する。しかも、バックグラウンドで`kubectl port-forward` コマンドの`pid`を含むファイルを作成する（ファイル名は`.portforward.pid`）。その後、`readinessProbe`の新しい`Authorization` ヘッダーを含む新しいelasticsearch client
yaml configuration に適用する。


### Connectionチェック

```bash
$ ./do.sh check_alive ${ENVIRONMENT}_password.txt
```

下記のログが表示すれば良い。

```bash
Waiting for the port to open...
Forwarding from 127.0.0.1:9200 -> 9200
Handling connection for 9200
{
  "name" : "es-client-869f4ddcb6-2zpm8",
  "cluster_name" : "es_linker",
  "cluster_uuid" : "uR-PxZ--SWqjOFoGhFAr0A",
  "version" : {
    "number" : "5.2.2",
    "build_hash" : "f9d9b74",
    "build_date" : "2017-02-24T17:26:45.835Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}
Closed port-forward (killed [.portforward.pid])
```

下記のログが続けて表示する場合、
`$ ./do.sh kill_portforwarding`を実行し、
`$ ./do.sh check_alive ${ENVIRONMENT}_password.txt`をもう一度実行してください。

```
curl: (7) Failed to connect to localhost port 9200: Connection refused
Waiting for the port to open...
curl: (7) Failed to connect to localhost port 9200: Connection refused
Waiting for the port to open...
curl: (7) Failed to connect to localhost port 9200: Connection refused
Waiting for the port to open...
curl: (7) Failed to connect to localhost port 9200: Connection refused
Waiting for the port to open...
curl: (7) Failed to connect to localhost port 9200: Connection refused
Waiting for the port to open...
curl: (7) Failed to connect to localhost port 9200: Connection refused
Waiting for the port to open...
```

---

>（翻訳保留中） Future Notes: 
> - Store password in secret
> - Change `readinessProbe` to `cmd` and access the password for the `Authorization` header from env vars