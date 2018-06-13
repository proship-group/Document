# Neo4j

## 概要

#### ファイル構成

```bash
neo4j
.
├── README.md
├── do.sh
└── neo4j.yaml
```

> 参考用

## nodeにラベルをつける
```
$ for no in "$(kubectl get nodes -l cloud.google.com/gke-nodepool=pool-db0 -o jsonpath='{.items[*].metadata.name}')"; do kubectl label no $no --overwrite hexalink.io/node-pool-role=database1 ; done

$ for no in "$(kubectl get nodes -l cloud.google.com/gke-nodepool=pool-db1 -o jsonpath='{.items[*].metadata.name}')"; do kubectl label no $no --overwrite hexalink.io/node-pool-role=database2 ; done

$ for no in "$(kubectl get nodes -l cloud.google.com/gke-nodepool=pool-rp0 -o jsonpath='{.items[*].metadata.name}')"; do kubectl label no $no --overwrite hexalink.io/node-pool-role=ingress ; done
```



## セットアップ

deployments directoryに移動する

```bash
$ cd ~/go/src/hexalink-k8s/middlewares/neo4j
```

### Node-Poolを選択

デフォルトでpodが`pool-db1` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**

### Deploy

```
$ kubectl create -f neo4j.yaml
```

下記のコマンドでコンポーネントを確認する

```
$ kubectl get pods -l component=neo4j
```
neo4jのPodのstatus が`running`になってから次のステップに進んでください。statusが`pending`であれば、`$ kubectl describe po <neo4jのpod名>`を実行し、`Events`部分を確認しpending問題を解決してください。


## Postセットアップ

### `neo4j` Userのパスワード変更


```
$ ./do.sh change_password [新しいパスワード, 現在のパスワード(デフォルトは「neo4j」）]]
```
>パスワードに`/`（スラッシュー）を設定しないでください。パスワードに`/`が含められば、apicoreをneo4jにコネクトする際にエラーが発生する。

上記のコマンドを実行し、下記のメッセージを表示すれば良い。

```
Handling connection for 7474
Password saved to 'password.txt'!

Closed port-forward (killed [.portforward.pid])
```

これで`${ENVIRONMENT}_password.txt`フォーマットのファイルがカレントディレクトリに作成される。その中に新しいパスワードを含める。必要な場合、そのファイルを切り取りしてください。

### Connection確認

```bash
$ ./do.sh check_alive ${ENVIRONMENT}_password.txt
```

下記のメッセイジを表示すれば良い。

```bash
Handling connection for 7474
{
  "extensions" : { },
  "node" : "http://localhost:7474/db/data/node",
  "relationship" : "http://localhost:7474/db/data/relationship",
  "node_index" : "http://localhost:7474/db/data/index/node",
  "relationship_index" : "http://localhost:7474/db/data/index/relationship",
  "extensions_info" : "http://localhost:7474/db/data/ext",
  "relationship_types" : "http://localhost:7474/db/data/relationship/types",
  "batch" : "http://localhost:7474/db/data/batch",
  "cypher" : "http://localhost:7474/db/data/cypher",
  "indexes" : "http://localhost:7474/db/data/schema/index",
  "constraints" : "http://localhost:7474/db/data/schema/constraint",
  "transaction" : "http://localhost:7474/db/data/transaction",
  "node_labels" : "http://localhost:7474/db/data/labels",
  "neo4j_version" : "3.3.3"
}
Closed port-forward (killed [.portforward.pid])
```

---

> （翻訳保留中）Future Notes:
> - Automate the changeing of password using k8s Secret and a container tool