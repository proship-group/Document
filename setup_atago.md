# atagoのセットアップ
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
### configctlとatagoのソースコード取得

```
$ mkdir -p ~/go/src && cd ~/go/src
$ git clone <configctl repository URL> -b <branch>
$ git clone <atago repository URL> -b <branch>
```

### configctlのビルドとデプロイ
### 設定ファイルのコピーと編集
```
$ cd ~/go/src/configctl
$ cp -pi config.env.example config.env
$ cp -pi database.yml.example database.yml
# atago用のconfigctlの設定ファイルはテンプレートをそのまま利用する
```

### ビルドとデプロイ
```
$ ./do.sh deploy_w_atago
```

## atagoのビルドとデプロイ
### do.shの変数設定
`SLACK_AUTH_TOKEN` `BOTID` `REMOTE_MACHINE_IP`を書き換えておく

### 設定ファイルのコピー
```
$ cd ~/go/src/atago
$ cp -pi config.env.example config.env
$ cp -pi deploy.yaml.example deploy.yaml
```

### config.envの編集
```
$ vi config.env
```
`SLACK_AUTH_TOKEN` `BOTID` `REMOTE_USER` `REMOTE_MACHINE_IP` `REMOTE_GOPATH` を書き換える。

### deploy.ymlの編集
環境毎の設定等を記載する
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

### 秘密鍵の設置
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
@atago deploylist
@atago k8s get po
```
