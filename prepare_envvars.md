# 環境変数設定

デプロイ前に以下の環境変数を設定してください。


***ターミナルを閉じたら環境変数の設定が消えるため、ターミナルを開くたびに必ず設定してください。***

```bash
# prodとstgのいずれかを設定してください。
$ export ENVIRONMENT=<prod/stg>

# GCPのコンソールで下記の情報を取得
$ export PROJECT_ID=<project id>
$ export CLUSTER=<cluster name>
$ export ZONE=<zone>
# 例: asia-northeast1-a
$ export GCS_BUCKET_NAME=<unique bucket name>

# 設定方法は「各種サービス設定」を参照
$ export SENTRY_DSN=<sentry.io/dsn>
# 例: https://78h8fh78234iuhsd8923:9y3hf37679h9ueh273hwh@sentry.io/738290

# ingressをデプロイするときに使う。
$ export INGRESS_DOMAIN=<domain.tld>

```

 設定例：

- prod環境

```
$ export PROJECT_ID=pit2-shared-prod
$ export ENVIRONMENT=prod
$ export CLUSTER=shared-prod
$ export ZONE=asia-northeast1-b
$ export SENTRY_DSN=https://b13b38bccaff4cab91d58a22db8c514d@sentry.io/300854
$ export INGRESS_DOMAIN=nextpit.tk
$ export GCS_BUCKET_NAME=pit2-shared-prod

```

- stg環境

```
$ export PROJECT_ID=pit2-shared-stg
$ export ENVIRONMENT=stg
$ export CLUSTER=shared-stg
$ export ZONE=asia-northeast1-b
$ export SENTRY_DSN=https://12a117b054d94114b2e1cc40e63e2591@sentry.io/304210
$ export INGRESS_DOMAIN=nextpit.ml
$ export GCS_BUCKET_NAME=pit2-shared-stg
```

---
設定後、`$ export`を実行して各変数の値を確認してください。
