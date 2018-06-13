# マイクロサービスのソースコード準備
## 必要なソースコード一覧
- configctl(別手順で取得済み)
- amagi
- linkermodels
- apicore
- notificator
- importer
- web-ui
- b-eee-lp
- mailfetcher
- taskManager
- linker-api
- actionScript
- adminTool
- hexalink-k8s

## atago経由の場合(まだ使えない)
Slackでatagoに対して指示（必要となる全ソースコード分繰り返す）
```
@atago clone_app <microservice git address>
```

## 手動の場合
### ソースコードのクローン
```
$ cd /Users/Administrator/Desktop/NextPit/enviroment/01_scripts/
$ ./01_clone_git_proship-group.sh

$ cd $GOPATH/src/
$ git clone git@github.com:proship/hexalink-k8s.git
```


## Clone Deployment Configurations

This repository contains the deployment files and scripts.

```bash
$ cd $GOPATH/src
$ git clone git@github.com:<Your repo>/k8s-deployments.git
```