
# MongoDB Sharded Cluster

## 概要 

こちらのファイルで3x3 sharded mongodb serverを作る.

#### ファイル構成

```bash 
mongodb
.
├── Dockerfile
├── README.md
├── configsvr.yaml
├── do.sh
├── mongos.yml
├── scripts
│   ├── config-svr-init.js
│   ├── rs-1-node-1-init.js
│   ├── rs-2-node-3-init.js
│   ├── rs-3-node-2-init.js
│   └── mongos-init.js
├── secrets
│   └── mongodb-keyfile
├── shards-node-1.yaml
├── shards-node-2.yaml
└── shards-node-3.yaml
```

> 参考用

## セットアップ

deployments directoryに移動する

```bash
$ cd $GOPATH/src/hexalink-k8s/middlewares/mongodb
```

### MongoDB Imageをビルド

（翻訳保留中）Due to current Kubernetes Volume Mounts limitation, we cannot set ownership of secret files (yet), we need to create a custom Mongodb image with the key-file packed inside already.

> GitHub Issue reference: https://github.com/kubernetes/kubernetes/issues/2630


#### `mongodb-keyfile`のkey-fileを作成

mongodb keyfileを作成する。後ほど作成するimageにコピーする。

```bash
$ openssl rand -base64 756 > secrets/mongodb-keyfile
```

#### imageビルド

こちらのスクリプトを実行することで、dockerをビルドし、プロジェクトのGCR repositoryに`gcloud docker -- push`する

```bash
$ ./do.sh build_push
```

### Node-Poolを選択

デフォルトでpodが`pool-db0` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**

### Deployment

scriptsのconfigmapを作成する

```bash
$ kubectl create configmap mongo-scripts --from-file=scripts/
```

sharded serverに全て必要なkubernetes resourcesを作成する

>`*.yaml`ファイルあるresourceが作成対象

```bash
$ ./do.sh deploy --all-yaml
```

下記のコマンドでcompoentをチェックする。podの起動が少し時間がかかるので、3分ほどお待ちください

```bash
$ kubectl get pods -l component=mongodb
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認

# shard components
$ kubectl get pods -l component=mongodb,role=mongoshard
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認

# config server components
$ kubectl get pods -l component=mongodb,role=mongoshard-cfgsvr
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認

# mongos components
$ kubectl get pods -l component=mongodb,role=mongos
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認

```

###`devops` パスワードを設定

```bash
$ ./do.sh change_password [設定したいパスワード, [パスワード保存先のファイル]]
```
`設定したいパスワード` に何も入力しなければ、このコマンドは自動的にランダムのパスワードを生成してくれる。
これで`${ENVIRONMENT}_password.txt`フォーマットのファイルがカレントディレクトリに作成される。その中に新しいパスワードが書いてある。必要な場合、そのファイルを切り取りしてください。


### Sharding設定

下記のコマンドは必ずPod起動後に実行してください。

```bash
$ ./do.sh init [パスワード保存先のファイル]
```

### MongDB稼働状況確認

```
# 1
$ cat ${ENVIRONMENT}_password.txt

# 2
$ kubectl get pods -l component=mongodb,role=mongos

# 3
$ kubectl exec -it <「# 2」で取得したいずれかのpod名>
# 「Welcome to the MongoDB shell.」を表示すれば「#4」に進めてください。表示しなければMongoDBとの接続に問題がある。手順を見直してください。

# 4
$ use admin

# 5
$ db.auth("devops","<「# 1」で取得した内容をここに入力>"
# 「Error: Authentication failed.」を表示すればMongDBの認証状況が正しくない。手順を見直してください。「1」を表示すれば「# 6」に進めてください。

# 6
$ sh.status()
# (確認すべき内容は要確認)

# 7
$ exit

```
