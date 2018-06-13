# GKEクラスタのセットアップ
## 環境設定
`devops`VMにログイン。ログインユーザを`devops`に変換。

```
$gcloud compute --project "pit2-shared-prod" ssh --zone "asia-northeast1-b" "devops"
$sudo su - devops 

```

kubectlコマンドを使えるようにする。
プロジェクト名とクラスタ名、クラスタzoneは、GCPのコンソールで取得
```
$ gcloud config set project <プロジェクト名>
$ gcloud container clusters get-credentials <クラスタ名> --zone <クラスタzone>
```
