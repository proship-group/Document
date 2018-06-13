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