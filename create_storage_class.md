# SSD Storage Class作成

## 概要

基本的に別の手順で作るデプロイメントは`fast` SSD Storage Classを使う。
このStorage Classは必ず別のマイクロサービスのデプロイを行う前にデプロイする。

#### ファイル構成

```bash
storageClass
.
└── fast.yaml
```

## セットアップ

`storageClass` directoryに移動

```bash
$ cd ~/go/src/hexalink-k8s/storageClass
```

下記のコマンドを実行する

```bash
$ ./do.sh deploy --all-yaml
```

下記のコマンドでStorage classの作成状況を確認


```bash
$ kubectl get storageclass fast

# should return something like
NAME      PROVISIONER            AGE
fast      kubernetes.io/gce-pd   1s
```