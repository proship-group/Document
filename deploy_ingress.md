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
# 「ERROR: (gcloud.compute.addresses.create) Could not fetch resource: The resource `XXXXX` already exists」を表示すれば無視してください。

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
$ cd ~/go/src/hexalink-k8s/middlewares/ingress
```

#### RBACが有効化された場合

ユーザを`cluster-admin`として初期化し、それで`nginx-controller`用のcluster service accountを作成する。

```bash
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)

```
`# 「Error from server (AlreadyExists): clusterrolebindings.rbac.authorization.k8s.io "cluster-admin-binding" already exists」`を表示すれば、「$　kubectl delete clusterrolebindings cluster-admin-binding」を実行してから上記のコマンドをもう一度実行してください。

> 参考: https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control#prerequisites_for_using_role-based_access_control

### ConfigMaps作成

`nginx-controller`用のConfigMapsを作成する

```bash
$ kubectl create -f configmaps/
```

### SSL作成

#### DNS設定変更

1. `GCPコンソール`-`pit-shared-prod`プロジェクト-`Compute Engine`-`VMインスタンス`で、`devops`の***外部IP***をメモする。
* [freenom](https://my.freenom.com)にアクセスする。ユーザ名とアカウントを入力しログインする。
* `Service`-`My Domains`をクリックする。
* prod環境であれば `nextpit.tk`行の`Manage Domain`をクリックする。stg環境であれば、`nextpit.ml`行の`Manage Domain`をクリックする。
* `Manage Freenom DNS`をクリックする。
* `Name`が「（空欄）」、「LP」、「API」、「ADM」行の、`Target`列に`手順1`で取得したIPを入力する。
* 画面の`Save Changes`ボタンを押す。DNS設定変更終了。

DNS設定の変更後、10～20分ほどでDNS情報が拡散される。その後、`devops`VMで下記のコマンドのいずれかを打てください。

```
# stg環境の場合
$ sudo /usr/local/certbot/certbot-auto certonly --standalone -d nextpit.ml -d lp.nextpit.ml -d api.nextpit.ml -d adm.nextpit.ml

# prod環境の場合
$ sudo /usr/local/certbot/certbot-auto certonly --standalone -d nextpit.tk -d lp.nextpit.tk -d api.nextpit.tk -d adm.nextpit.tk
```
`IMPORTANT NOTES`の内容をメモしてください。そこに`cert`ファイル(fullchain.pem)と`key`ファイル(privkey.pem)の保存先が書いてある。

#### .pemを変換

```
# スーパーユーザに変更
$ sudo su
$ cd <certファイルとkeyファイルの保存先>

# .pemを.keyに変換（pemエンコーディング）
$ sudo openssl rsa -outform pem -in privkey.pem -out privkey.key

# .pemを.certに変換（pemエンコーディング）
sudo openssl x509 -outform pem -in fullchain.pem -out fullchain.crt

# スーパーユーザをログアウト
$ exit
```

作成した`privkey.pem`と`fullchain.crt`のディレクトリをメモしてください。
>参考
>
>https://stackoverflow.com/questions/19979171/how-to-convert-pem-into-key
>
>https://code.i-harness.com/ja/q/d18bda


### TLSファイルをSecretとして設定

下記のコマンドを実行することでSSLを取得しsecretファイルにそれを追加する
#### **! ! ! ! 必ずSSLのディレクトリに移動してから実行 ! ! ! !**

```bash
$ cd <certファイルとkeyファイルの保存先>
$ kubectl sudo kubectl create secret tls linker-ssl --key <privkey.keyのディレクトリ> --cert <fullchain.crtのディレクトリ>
```

> `Permission Denied`エラーが発生すれば、上記のコマンドの代わりに下記のコマンドを実行してください。

```bash
$ sudo kubectl create secret tls linker-ssl --key <privkey.keyのディレクトリ> --cert <fullchain.crtのディレクトリ>
```

## Deployment

deployments directoryに移動

```
$ cd ~/go/src/hexalink-k8s/middlewares/ingress
```

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

**`LOADBALANCER_IP`が正しく設定することを確認してから実行してください。**

**LoadBalancerをデプロイする**

GCE LoadBalancer を作成する

```bash
$ ./do.sh deploy loadbalancer.yml
```

## DNS設定戻す

1. 「$ kubectl get svc -l role=loadbalancer」を実行し、出力した`EXTERNAL-IP`をメモする。
* [freenom](https://my.freenom.com)にアクセスする。ユーザ名とアカウントを入力しログインする。
* `Service`-`My Domains`をクリックする。
* prod環境であれば `nextpit.tk`行の`Manage Domain`をクリックする。stg環境であれば、`nextpit.ml`行の`Manage Domain`をクリックする。
* `Manage Freenom DNS`をクリックする。
* `Name`が「（空欄）」、「LP」、「API」、「ADM」行の、`Target`列に`手順1`で取得したIPを入力する。
* 画面の`Save Changes`ボタンを押す。DNS設定変更終了。

DNS設定の変更後、10～20分ほどでDNS情報が拡散される。その後、下記のいずれかのリンクをクリックしてください。環境構築が問題なければ、Linkerのログイン画面が表示される。

1. [prod環境の場合](nextpit.tk)
2. [stg環境の場合](nextpit.ml) 




