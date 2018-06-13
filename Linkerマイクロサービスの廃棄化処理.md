#環境の廃棄処理

##事前準備
`devops`VMにアクセスし、`devops`ユーザとしてログイン

```
$ gcloud compute --project "pit2-shared-prod" ssh --zone "asia-northeast1-b" "devops"

$ sudo su - devops
```


[環境変数設定](prepare_envvars.md)と`devops`VMのアクセス先（stg環境かprod環境）設定を行う。

---

###`devops`VMをstg環境にアクセスする場合

```
$ gcloud config set project pit2-shared-stg
$ gcloud container clusters get-credentials shared-stg --zone asia-northeast1-b
```
下記のメッセージが表示すれば、stgに正しくアクセスできる
```
kubeconfig entry generated for shared-stg.
```

---

###`devops`VMをprod環境にアクセスする場合

```
$ gcloud config set project pit2-shared-prod
$ gcloud container clusters get-credentials shared-prod --zone asia-northeast1-b
```
下記のメッセージが表示すれば、stgに正しくアクセスできる
```
kubeconfig entry generated for shared-prod.
```

---



##ミドルウェアの廃棄化処理

###MongoDBの廃棄化処理
```
$ cd ~/go/src/hexalink-k8s/middlewares/mongodb

$ kubectl delete -f mongos.yml
$ kubectl delete -f shards-node-1.yaml
$ kubectl delete -f shards-node-2.yaml
$ kubectl delete -f shards-node-3.yaml
$ kubectl delete -f configsvr.yaml

# pvcを削除
$ kubectl delete pvc db-cfg-mongo-config-svr-0 db-cfg-mongo-config-svr-1 db-cfg-mongo-config-svr-2 db-rs-1-mongo-shard-node-1-0 db-rs-1-mongo-shard-node-2-0 db-rs-1-mongo-shard-node-3-0 db-rs-2-mongo-shard-node-1-0 db-rs-2-mongo-shard-node-2-0 db-rs-2-mongo-shard-node-3-0 db-rs-3-mongo-shard-node-1-0 db-rs-3-mongo-shard-node-2-0 db-rs-3-mongo-shard-node-3-0   

```


実行結果確認

```
$ kubectl get svc
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い

$ kubectl get statefulset
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い

$ kubectl get deployment
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い

$ kubectl get pvc
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い

$ kubectl get pods 
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

$ kubectl get configmap
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete configmap <configmap名>」で削除してください。

```


###MySQLの廃棄化処理

```
$ cd ~/go/src/hexalink-k8s/middlewares/mysql

$ kubectl delete -f cm-mysql.yaml
$ kubectl delete -f mysql.yaml
$ kubectl delete -f svc-mysql.yaml

# pvc削除
$ kubectl delete pvc data-mysql-0 data-mysql-1 data-mysql-2

```

実行結果確認

```
$ kubectl get configmap
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い

$ kubectl get statefulset
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い

$ kubectl get service
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い

$ kubectl get pods 
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

$ kubectl get configmap
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete configmap <configmap名>」で削除してください。

```

###Elasticsearchの廃棄化処理


```
$ cd ~/go/src/hexalink-k8s/middlewares/elasticsearch

$ kubectl delete -f es-data-svc.yaml
$ kubectl delete -f es-master.yaml 
$ kubectl delete -f es-data-stateful.yaml
$ kubectl delete -f es-discovery-svc.yaml
$ kubectl delete -f es-svc.yaml

# es-client を削除
$ cd ~/go/src/hexalink-k8s/middlewares/elasticsearch/es-client
$ kubectl delete -f ${ENVIRONMENT}.yaml

```

実行結果確認

```
$ kubectl get service
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い

$ kubectl get deployment
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い

$ kubectl get statefulset
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い

$ kubectl get pods 
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

$ kubectl get configmap
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete configmap <configmap名>」で削除してください。

```

###redis廃棄化処理
```
$ cd ~/go/src/hexalink-k8s/middlewares/redis

# pvc削除
$ kubectl delete -f pvc.yaml 

$ kubectl delete -f redis-single.yaml 

$ kubectl delete -f secret.template.yaml

```
実行結果確認

```
$ kubectl get pvc
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い

$ kubectl get service
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い

$ kubectl get deployment
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い

$ kubectl get pods 
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

```

###neo4jの廃棄化処理

```
$ cd ~/go/src/hexalink-k8s/middlewares/neo4j

$ kubectl delete -f neo4j.yaml 

```

実行結果確認

```
$ kubectl get pvc
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い

$ kubectl get service
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い

$ kubectl get deployment
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い

$ kubectl get pods 
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

```

###NSQの廃棄化処理

```
$ cd ~/go/src/hexalink-k8s/middlewares/nsq
$ kubectl delete -f service

```

実行結果確認

```
$ kubectl get service
#実行結果に対して「NAME」に「nsq」が付いているものがなければ良い

$ kubectl get deployment
#実行結果に対して「NAME」に「nsq」が付いているものがなければ良い

$ kubectl get pods 
#実行結果に対して「NAME」に「nsq」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

```


## Linkerマイクロサービスの廃棄化処理



