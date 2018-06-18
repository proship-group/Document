# NSQ Server

## 概要

こちらのconfigurationファイルはシングルのNSQサーバを作成する


#### ファイル構成

```bash
nsq
.
└── nsq.yml
```

> 参考用

## セットアップ

deployments directoryに移動する

```bash
$ cd ~/go/src/hexalink-k8s/middlewares/nsq
```

### Node-Poolを選択

デフォルトでpodが`pool-db0` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**


### Deployment

下記のコマンドを実行し、NSQサーバをデプロイする

```bash
$ kubectl create -f nsq.yaml
```
下記のコマンドでcomponentを確認する。podの起動が少し時間がかかるので、3分ほどお待ちください。

```
$ kubectl get pods -l  component=nsq
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認
```