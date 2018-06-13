# 環境変数設定

デプロイ前に以下の環境変数を設定してください。


※ターミナルを閉じたら環境変数の設定が消えるため、ターミナルを開くたびに必ず設定してください。

```bash
# prodとstgのいずれかを設定してください。
$ export ENVIRONMENT=<prod/stg>

#GCPのコンソールで下記の情報を取得
$ export PROJECT_ID=<project id>
$ export CLUSTER=<cluster name>
$ export ZONE=<zone>
# 例: asia-northeast1-a
$ export GCS_BUCKET_NAME=<unique bucket name>

#設定方法は「各種サービス設定」を参照
$ export SENTRY_DSN=<sentry.io/dsn>
# 例: https://78h8fh78234iuhsd8923:9y3hf37679h9ueh273hwh@sentry.io/738290

# ingressをデプロイするときに使う。
$ export INGRESS_DOMAIN=<domain.tld>

```
