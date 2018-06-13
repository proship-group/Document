# Redis

## 概要

#### ファイル構成

```bash
redis
.
├── README.md
├── config
│   └── redis.conf
├── do.sh
├── pvc.yaml
├── secret.template.yaml
└── redis-single.yml
```

> 参考用

## セットアップ

deployments directoryに移動する

```bash
$ cd ~/go/src/k8s-deployments/middlewares/redis
```

### Password作成

ランダムパスワードを作成するのに下記のコマンドを実行する

```bash
$ ./do.sh change_redis_password
```

このコマンドを実行することでカレントディレクトに新しいパスワードを含むファイルを作成する（ファイル名のフォーマットは`${ENVIRONMENT}_password.txt`）。必要な場合、それを切り取りしてください。

ランダムではないパスワードの作成も可能。
（作成方法要調査）


#### Secret作成

作成したPasswordを使いredisのconfigurationファイルを作成する。 それを作成するために、下記のコマンドを実行する。

```bash
$ ./do.sh create_redis_config [パスワードもしくはパスワードファイル]
```

> パスワードファイルを使う場合、このスクリプトは自動的にファイルのコンテンツをパスワードとして使う。
> 
> パスワードファイルにスペースは必ずないように確認してください。


### Node-Pool選択

デフォルトでpodが`pool-db0` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**


### Deployment

まず、redis用の`PersistentVolumeClaim`を作る

```bash
$ kubectl create -f pvc.yaml
```

redisサーバーをデプロイする

```bash
$ kubectl create -f redis-single.yaml
```