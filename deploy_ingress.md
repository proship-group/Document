# Ingress

## 概要

#### ファイル構成

```bash
ingress
.
├── configmaps
│   ├── nginx-settings-configmap.yml
│   └── sticky-sessions.yml
├── do.sh -> ../../do.sh
├── glbc.yml
├── ingress.yml
├── loadbalancer.yml
└── rbac.yml
```

## セットアップ

#### Static IPアドレスの作成

まず、GCP LoadBalancer用のstatic IPアドレスを作成する

```bash
$ gcloud compute addresses create ${PROJECT_ID} --region=$(awk -F'-' '{a=$1"-"$2; print a}' <<< ${ZONE})
```

下記のコマンドでIPを取得する

```bash
$ gcloud compute addresses list --filter="name=( '${PROJECT_ID}' )" --format="get('address')"
```

下記の環境変数を追加する

```bash
$ export LOADBALANCER_IP=xxx.xxx.xxx.xxx

# onliner set
$ export LOADBALANCER_IP=$(gcloud compute addresses list --filter="name=( '${PROJECT_ID}' )" --format="get('address')")
# check value
$ echo $LOADBALANCER_IP
```


deployments directoryに移動

```bash
$ cd ~/go/src/k8s-deployments/middlewares/ingress
```

#### RBACが有効化された場合

ユーザを`cluster-admin`として初期化し、それで`nginx-controller`用のcluster service accountを作成する。

```bash
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)
```
> 参考: https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control#prerequisites_for_using_role-based_access_control

### ConfigMaps作成

`nginx-controller`用のConfigMapsを作成する

```bash
$ kubectl create -f configmaps/
```

### TLSファイルをSecretとして設定

下記のコマンドを実行することでSSLを取得しsecretファイルにそれを追加する
#### **! ! ! ! 必ずSSLのディレクトリに移動してから実行 ! ! ! !**

```bash
$ kubectl create secret tls linker-ssl --key <TLS.keyのディレクトリ> --cert <TLS.crtのディレクトリ>
```

> `Permission Denied`エラーが発生すれば、上記のコマンドを使わずに下記のコマンドを実行してください。

```bash
$ sudo kubectl create secret tls linker-ssl --key <TLS.keyのディレクトリ> --cert <TLS.crtのディレクトリ>
```

## Deployment

#### Nginx-Controller用のService Accountを作成する

下記のコマンドを実行することで、`nginx-controller`に必要なpermissionを持つロールを作成する。しかもそれにpod用のservice accountをbindする。


```bash
$ kubectl create -f rbac.yml
```

#### Ingress resource作成

下記のk8s resourceを順番に作成.

```bash
$ kubectl create -f glbc.yml
$ ./do.sh deploy ingress.yml
```

### Create the GCE LoadBalancer

**`LOADBALANCER_IP`が正しく設定することを確認してから実行してください。 **

**LoadBalancerをデプロイする**

GCE LoadBalancer を作成する

```bash
$ ./do.sh deploy loadbalancer.yml
```

