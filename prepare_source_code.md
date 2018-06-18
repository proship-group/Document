# マイクロサービスのソースコード準備
## 必要なソースコード一覧
- configctl
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
$ cd ~/go/src
$ cat << EOS | while read LINE; do git clone git@github.com:<Your repo>/${LINE}.git -b <Your branch>; done
linkermodels
apicore
notificator
importer
web-ui
b-eee-lp
mailfetcher
taskManager
linker-api
actionScript
adminTool
EOS
```
### amagiのクローン
amagiは `go get` 時にmasterリポジトリから取得してしまうため、master以外のリポジトリから取得したい場合にのみ実施。
```
$ mkdir -p ~/go/src/github.com/b-eee && cd ~/go/src/github.com/b-eee
$ git clone git@github.com:b-eee/amagi.git -b <your branch>
```
### 依存関係の解消
golang以外のリポジトリではエラーになりますが、無視してください。
```
$ cd ~/go/src && ls -1 | while read LINE; do cd ~/go/src/$LINE && go get -v ./...; done
```

## Clone Deployment Configurations

This repository contains the deployment files and scripts.

```bash
$ cd ~/go/src
$ git clone git@github.com:<Your repo>/k8s-deployments.git
```

## Clone Deployment Configurations

This repository contains the deployment files and scripts.

```bash
$ cd $GOPATH/src
$ git clone git@github.com:<Your repo>/k8s-deployments.git
```