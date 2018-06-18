# atagoのセットアップ

本手順の実行前提：`configctl`のデプロイは正常終了。

## devops-vmとのアクセス確認
devops-vmとのアクセスを確認する。
`devops`vmにアクセスできれば、
`devops-vm作成と設定変更`と`atago-vmのOS設定`のステップをスキップしてください。アクセスの確認方法は、下記のコマンドを実行してください。

```
$ gcloud compute --project "pit2-shared-prod" ssh --zone "asia-northeast1-b" "devops"

```

上記のコマンドを実行後に下記のメッセージを表示すれば
`devops`VMにアクセスできた。`devops-vm作成と設定変更`と`atago-vmのOS設定`のステップをスキップしてください。

```
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-1011-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

89 packages can be updated.
0 updates are security updates.


*** System restart required ***

```
`devops`VMとのアクセスを切る。

```
$ exit
```

## devops-vm作成と設定変更

### キーペアを作成
ローカルで作業

```
# 秘密鍵のpassphraseは設定しない。（Enterを押下）
$ ssh-keygen -f ~/.ssh/devops_rsa -t rsa -b 4096 -C "devops"
# 上記の公開鍵をクリップボードにコピーしておく。
$ cat ~/.ssh/devops_rsa.pub
```

### VM作成
1. Google Cloud Consoleから以下のように画面遷移
    - `Compute Engine` -> `VM instances`
2. 画面上部の `CREATE INSTANCE` を押下
3. 以下のスペックでVMを作成する
   - vm name -> `devops-vm`
   - machine type -> `g1-small`
   - zone -> `asia-northeast1-b`
   - os -> `Ubuntu 16.04 LTS`
   - disk -> `20GB standard persistent disk(boot disk)`
   - `Enable deletion protection`
   - SSH keys -> 上記で作成した公開鍵を貼り付け

### 固定IPに変更
1. Google Cloud Consoleから以下のように画面遷移
   - `VPC network` -> `External IP addresses`
2. `devops-vm` のIPアドレスを `static` に変更
   - グローバルIPアドレスをメモしておく

### devops-vmにSSHログイン
ローカルで作業

```
$ ssh -i ~/.ssh/devops_rsa devops@<IPアドレス>
```

## atago-vmのOS設定
ここからリモート作業。
### OSのタイムゾーン設定

```
$ sudo timedatectl set-timezone Asia/Tokyo
$ date
```

### OSのアップデート 

```
$ sudo apt-get update
$ sudo apt-get upgrade
```


## ミドルウェアのインストールと設定
ここから`devops`VMに`devops`ユーザとしてログインしてから進めてください。

下記のコマンドを実行し`devops`VMにアクセスし、`devops`ユーザとしてログインする。

```
$ gcloud compute --project "pit2-shared-prod" ssh --zone "asia-northeast1-b" "devops"
$ sudo su - devops
```

### Google Cloud SDKとkubectlのインストール確認

下記のコマンドを実行してください。

```
$ gcloud version
$ kubectl version
```
バージョン情報を表示すれば、
`Google Cloud SDKとkubectlのインストール`ステップをスキップしてください。

`No command`メッセージが表示すれば、
`Google Cloud SDKとkubectlのインストール`ステップに進めてください。

### Google Cloud SDKとkubectlのインストール
ref) https://cloud.google.com/sdk/downloads

```
# 変数設定
$ export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"

# リポジトリ追加
$ echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

# 鍵のインポート
$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Google Cloud SDKのインストール
$ sudo apt-get update && sudo apt-get install google-cloud-sdk

# kubectlのインストール
$ sudo apt-get install kubectl

# gcloudコマンド初期化
$ gcloud init
# 表示内容に沿って進める
```

###  Docker CEのインストール確認

下記のコマンドを実行してください。

```
$ docker version
```
バージョン情報を表示すれば、
`Docker CEのインストール`ステップをスキップしてください。

`No command`メッセージが表示すれば、
`Docker CEのインストール`ステップに進めてください。

###  Docker CEのインストール
ref) https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#supported-storage-drivers

```
# 依存パッケージのインストール
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# 鍵のインポート
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# fingerprintの確認 ("9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88" が表示されればOK)
$ sudo apt-key fingerprint 0EBFCD88

# リポジトリ追加
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# Docker CEのインストール
$ sudo apt-get update
$ sudo apt-get install docker-ce

# テスト
$ sudo docker run hello-world

# dockerグループにdevopsユーザーを追加
$ sudo usermod -aG docker devops
```

### golangのインストール確認

下記のコマンドを実行してください。

```
$ go version
```
バージョン情報を表示すれば、
`golangのインストール`ステップをスキップしてください。

`No command`メッセージが表示すれば、
`golangのインストール`ステップに進めてください。


### golangのインストール

```
# golangのダウンロード
$ curl -OL https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz

# インストール
$ sudo tar -C /usr/local -zxf go1.9.3.linux-amd64.tar.gz
$ vi ~/.bash_profile
末尾に以下を追記
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin

# 確認
$ exec -l $SHELL
$ go version
```

### npm, bowerのインストール確認
下記のコマンドを実行してください。

```
$ npm version
$ bower version
```
バージョン情報を表示すれば、
`npm, bowerのインストール`ステップをスキップしてください。

`No command`メッセージが表示すれば、
`npm, bowerのインストール`ステップに進めてください。


### npm, bowerのインストール

```
# リポジトリの追加
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

# nodejsのインストール
$ sudo apt-get install nodejs

# nodeコマンドとnodejsの関連付け
$ sudo update-alternatives --install /usr/bin/node node /usr/bin/nodejs 10

# bowerとgulpのインストール
$ sudo npm install -g bower
$ sudo npm install -g gulp

# その他追加パッケージのインストール
$ sudo apt-get install -y python make g++
$ sudo npm install -g node-gyp
$ sudo npm install -g sse4_crc32
```

## atagoのビルドとデプロイ
### atagoのソースコード取得

```
$ mkdir -p ~/go/src && cd ~/go/src
$ git clone <atago repository URL> -b <branch>
```

### atago-configctlのビルドとデプロイ

```
$ cd ~/go/src/configctl
$ ./do.sh deploy_w_atago
```

 `SLACK_AUTH_TOKEN ` `BOTID` `REMOTE_USER` `REMOTE_MACHINE_IP` `REMOTE_GOPATH` のいずれも変更がなければ、 `config.envファイルの設定` ステップをスキップしてください。（黄：変更したかどうかの確認方法は要追加。）


atago経由でデプロイ可能な対象の変更がなければ、`deploy.yamlファイルの設定` ステップをスキップしてください。デプロイ対象の変更有無に関しては開発者に確認してください。

---
### config.envファイルの設定

#### 1. config.envファイルのバックアップ取得

```
$ cd ~/go/src/atago

# 既存設定ファイルのバックアップ取得
$ cp config.env config.env.bk.`date "+%Y%m%d_%I%M%S"`
# 「No such file or directory」を表示しても無視してください。

```

#### 2. config.envの編集

```
# 設定ファイルコピー
$ cp -pi config.env.example config.env
# 「cp: overwrite 'config.env'?」を表示すると「yes」を入力

$ vi config.env
```
`SLACK_AUTH_TOKEN` `BOTID` `REMOTE_USER` `REMOTE_MACHINE_IP` `REMOTE_GOPATH` を書き換える。

---

### deploy.yamlファイルの設定

#### 1.  deploy.yamlファイルのバックアップ取得
```
$ cd ~/go/src/atago

# 既存設定ファイルのバックアップ取得
$ cp deploy.yaml deploy.yaml.bk.`date "+%Y%m%d_%I%M%S"`
# 「No such file or directory」を表示しても無視してください。
```


#### 2. deploy.yamlの編集

```
# 設定ファイルコピー
$ cp -pi deploy.yaml.example deploy.yaml
# 「cp: overwrite 'deploy.yaml'?」を表示する。 「yes」を入力

$ vi deploy.yaml
```

環境毎の設定等を記載する。

```
# Example
environments:
  - name: stg
    targets:
      - name: configctl
        repository:
          name: configctl
          branch: develop
        dockerimage:
          name: configctl
          npmbowerinstall: false
          deployment: configctl
          container: configctl
      - name: apicore
        repository:
          name: apicore
          branch: develop
        dockerimage:
          name: apicore
          npmbowerinstall: false
          deployment: apicore
          container: apicore
        dependencies:
          - linkermodels
      - name: notificator
        repository:
          name: notificator
          branch: develop
        dockerimage:
          name: notificator
          npmbowerinstall: false
          deployment: notificator
          container: notificator
      - name: importer
        repository:
          name: importer
          branch: develop
        dockerimage:
          name: importer
          npmbowerinstall: false
          deployment: importer
          container: importer
        dependencies:
          - linkermodels
      - name: web-ui
        repository:
          name: web-ui
          branch: develop
        dockerimage:
          name: ui
          npmbowerinstall: true
          deployment: web-ui
          container: ui
      - name: b-eee-lp
        repository:
          name: b-eee-lp
          branch: develop
        dockerimage:
          name: b-eee-lp
          npmbowerinstall: true
          deployment: b-eee-lp
          container: b-eee-lp
      - name: mailfetcher
        repository:
          name: mailfetcher
          branch: develop
        dockerimage:
          name: mailfetcher
          npmbowerinstall: false
          deployment: mailfetcher
          container: mailfetcher
      - name: taskmanager
        repository:
          name: taskManager
          branch: develop
        dockerimage:
          name: taskmanager
          npmbowerinstall: false
          deployment: taskmanager
          container: taskmanager
      - name: linker-api
        repository:
          name: linker-api
          branch: develop
        dockerimage:
          name: linker-api
          npmbowerinstall: false
          deployment: linker-api
          container: linker-api
      - name: actionscript
        repository:
          name: actionScript
          branch: develop
        dockerimage:
          name: actionscript
          npmbowerinstall: true
          deployment: actionscript
          container: actionscript
    gke:
      name: linker-test
      project: onigiri-8441
      cluster: linker-test
      zone: asia-northeast1-c
```
>備考:
>atagoでデプロイ可能なマイクロサービスはdeploy.yamlに制御される。

---

### 秘密鍵の設置

前述`devops-vm作成と設定変更`の「VM作成」ステップは行わなければ、このステップはスキップしてください。`ビルドとデプロイ`ステップに進めてください。

```
$ cd ~/go/src/atago/ssh_keys
$ vi id_rsa
# VM作成で作成したキーペアの秘密鍵をペーストする
```

### ビルドとデプロイ

```
$ cd ~/go/src/atago
$ ./do.sh build_linux
$ ./do.sh deploy_atago
```

### テスト
SlackからBotに対して以下のメッセージを送る

```
/atago deploylist
/atago k8s get po
```
