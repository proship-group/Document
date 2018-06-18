# 環境の廃棄処理

## 事前準備
`devops`VMにアクセスし、`devops`ユーザとしてログイン

```
$ gcloud compute --project "pit2-shared-prod" ssh --zone "asia-northeast1-b" "devops"
$ sudo su - devops
```


[環境変数設定](prepare_envvars.md)と`devops`VMのアクセス先（stg環境かprod環境）の設定を行う。

***`devops`VMのアクセス先を間違ったら別環境を壊すことになるため、慎重に確認してください。***

---

### `devops`VMをstg環境にアクセスする場合
下記のコマンドを実行してください。

```
$ gcloud config set project pit2-shared-stg
$ gcloud container clusters get-credentials shared-stg --zone asia-northeast1-b
```
下記のメッセージが表示すれば、stgに正しくアクセスできた
```
kubeconfig entry generated for shared-stg.
```

---

### `devops`VMをprod環境にアクセスする場合
下記のコマンドを実行してください。

```
$ gcloud config set project pit2-shared-prod
$ gcloud container clusters get-credentials shared-prod --zone asia-northeast1-b
```
下記のメッセージが表示すれば、prodに正しくアクセスできた
```
kubeconfig entry generated for shared-prod.
```

---



## ミドルウェアの廃棄化処理

### MongoDBの廃棄化処理
```
$ cd ~/go/src/hexalink-k8s/middlewares/mongodb

$ kubectl delete -f mongos.yml
$ kubectl delete -f shards-node-1.yaml
$ kubectl delete -f shards-node-2.yaml
$ kubectl delete -f shards-node-3.yaml
$ kubectl delete -f configsvr.yaml

# pvc削除
$ kubectl delete pvc db-cfg-mongo-config-svr-0 db-cfg-mongo-config-svr-1 db-cfg-mongo-config-svr-2 db-rs-1-mongo-shard-node-1-0 db-rs-1-mongo-shard-node-2-0 db-rs-1-mongo-shard-node-3-0 db-rs-2-mongo-shard-node-1-0 db-rs-2-mongo-shard-node-2-0 db-rs-2-mongo-shard-node-3-0 db-rs-3-mongo-shard-node-1-0 db-rs-3-mongo-shard-node-2-0 db-rs-3-mongo-shard-node-3-0   

```


実行結果確認

```
$ kubectl get svc
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば「$ kubectl delete svc <svc名>」で削除してください。

$ kubectl get statefulset
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば「$ kubectl delete statefulset <statefulset名>」で削除してください。

$ kubectl get deployment
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば「$ kubectl delete deployment <deployment名>」で削除してください。

$ kubectl get pvc
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば「$ kubectl delete pvc <pvc名>」で削除してください。

$ kubectl get pods 
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

$ kubectl get configmap
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete configmap <configmap名>」で削除してください。

```


### MySQLの廃棄化処理

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
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い。あれば「$ kubectl delete configmap <configmap名>」で削除してください。


$ kubectl get statefulset
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い。あれば「$ kubectl delete statefulset <statefulset名>」で削除してください。


$ kubectl get service
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い。あれば「$ kubectl delete svc <service名>」で削除してください。


$ kubectl get pods 
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「mysql」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

```

### Elasticsearchの廃棄化処理


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
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete service <service名>」で削除してください。

$ kubectl get deployment
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete deployment <deployment名>」で削除してください。

$ kubectl get statefulset
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete statefulset <statefulset名>」で削除してください。

$ kubectl get pods 
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

$ kubectl get configmap
#実行結果に対して「NAME」に「mongo」が付いているものがなければ良い。あれば、「$ kubectl delete configmap <configmap名>」で削除してください。

```

### redis廃棄化処理
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
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い。あれば、「$ kubectl delete pvc <pvc名>」で削除してください。

$ kubectl get service
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い。あれば、「$ kubectl delete service <service名>」で削除してください。

$ kubectl get deployment
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い。あれば、「$ kubectl delete deployment <deployment名>」で削除してください。

$ kubectl get pods 
#実行結果に対して「NAME」に「es」もうしくは「elasticsearch」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「redis」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

```

### neo4jの廃棄化処理

```
$ cd ~/go/src/hexalink-k8s/middlewares/neo4j

$ kubectl delete -f neo4j.yaml 

```

実行結果確認

```
$ kubectl get pvc
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い。あれば、「$ kubectl delete pvc <pvc名>」で削除してください。

$ kubectl get service
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い。あれば、「$ kubectl delete service <service名>」で削除してください。

$ kubectl get deployment
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い。あれば、「$ kubectl delete deployment <deployment名>」で削除してください。

$ kubectl get pods 
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

$ kubectl get secret
#実行結果に対して「NAME」に「neo4j」が付いているものがなければ良い。あれば、「$ kubectl delete secret <secret名>」で削除してください。

```

### NSQの廃棄化処理

```
$ cd ~/go/src/hexalink-k8s/middlewares/nsq
$ kubectl delete -f service

```

実行結果確認

```
$ kubectl get service
#実行結果に対して「NAME」に「nsq」が付いているものがなければ良い。あれば、「$ kubectl delete service <service名>」で削除してください。

$ kubectl get deployment
#実行結果に対して「NAME」に「nsq」が付いているものがなければ良い。あれば、「$ kubectl delete deployment <deployment名>」で削除してください。

$ kubectl get pods 
#実行結果に対して「NAME」に「nsq」が付いているものがなければ良い。あれば、「$ kubectl delete pods <pod名>」で削除してください。

```

### Ingressの廃棄化処理

```
$ cd ~/go/src/hexalink-k8s/middlewares/ingress
$ kubectl delete -f glbc.yml
$ kubectl delete -f loadbalancer.yml
$ kubectl delete -f rbac.yml
```

実行結果確認

```
$ kubectl get configmaps
# 実行結果に対して「ingress-controller-leader-nginx」以外、「NAME」に「nginx1」か「ingress」が付いているものがなければ良い。あれば、「$ kubectl delete configmaps <configmaps名>」で削除してください。

$ kubectl get secrets
# 実行結果に対して「NAME」に「linker-ssl」が付いているものがなければ良い。あれば、「$ kubectl delete secrets <secrets名>」で削除してください。

$ kubectl get service
# 実行結果に対して「NAME」に「default-http-backend」か「loadbalancer」が付いているものがなければ良い。あれば、「$ kubectl delete service <service名>」で削除してください。

$ kubectl get deployment
# 実行結果に対して「NAME」に「default-http-backend」か「nginx-ingress」が付いているものがなければ良い。あれば、「$ kubectl delete deployment <deployment名>」で削除してください。

$ kubectl get serviceaccount
# 実行結果に対して「NAME」に「nginx-ingress」が付いているものがなければ良い。あれば、「$ kubectl delete serviceaccount <serviceaccount名>」で削除してください。

$ kubectl get clusterrole
# 実行結果に対して「NAME」に「nginx-ingress」が付いているものがなければ良い。あれば、「$ kubectl delete clusterrole <clusterrole名>」で削除してください。

$ kubectl get clusterrole
# 実行結果に対して「NAME」に「nginx-ingress」が付いているものがなければ良い。あれば、「$ kubectl delete role <role名>」で削除してください。

$ kubectl get rolebinding
# 実行結果に対して「NAME」に「nginx-ingress」が付いているものがなければ良い。あれば、「$ kubectl delete rolebinding <rolebinding名>」で削除してください。

$ kubectl get clusterrolebinding
# 実行結果に対して「NAME」に「nginx-ingress」が付いているものがなければ良い。あれば、「$ kubectl delete clusterrolebinding <clusterrolebinding名>」で削除してください。

```


## Linkerマイクロサービスの廃棄化処理

### configctlの廃棄化処理

```
$ kubectl get deployment
#実行結果に対して「NAME」に「configctl」が付いているものがなければ良い。あれば「$ kubectl delete deploy <deployment名>」で削除してください。

$ kubectl get svc
#実行結果に対して「NAME」に「configctl」が付いているものがなければ良い。あれば「$ kubectl delete svc <service名>」で削除してください。

$ kubectl get po
#実行結果に対して「NAME」に「configctl」が付いているものがなければ良い。あれば「$ kubectl delete po <pod名>」で削除してください。
```

### apicoreの廃棄化処理
```
$ kubectl get deployment
#実行結果に対して「NAME」に「apicore」が付いているものがなければ良い。あれば「$ kubectl delete deploy <deployment名>」で削除してください。

$ kubectl get svc
#実行結果に対して「NAME」に「apicore」が付いているものがなければ良い。あれば「$ kubectl delete svc <service名>」で削除してください。

$ kubectl get po
#実行結果に対して「NAME」に「apicore」が付いているものがなければ良い。あれば「$ kubectl delete po <pod名>」で削除してください。
```


