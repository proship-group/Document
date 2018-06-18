
# 各種サービス設定
## Google Cloud Platform
apicoreとmailfetcherで使用するオブジェクトストレージとアカウントを作成する。
### Service accountの作成
1. 以下のURLにアクセス
   - https://console.cloud.google.com/apis/credentials
2. `Create credentials` -> ` Service account key` の順に押下
3. 以下の順に操作
   - Service account -> `New service account`
   - Service account name -> 任意
   - Role -> `Storage/Storage Object Creator`
   - Key type -> `JSON`
   - `Create` を押下
4. JSONファイルがダウンロードされる
> 権限が強すぎるため、後日見直しが必要と思われる

### Google Cloud Storageバケットの作成
1. 以下のURLにアクセス
   - https://console.cloud.google.com/storage
2. `CREATE BUCKET` を押下
3. 以下の通り設定し、`Create` を押下する
   - Name -> 任意
   - Default storage class -> `Regional`
   - Regional location -> クラスタと同じリージョン

- - -


## Sendgrid
LINKERからの各種メール送付に使用。

### API Keyの発行
1. Webブラウザで以下のURLにアクセスしログインする
   - https://sendgrid.com/
2. `Settings` -> `API Keys` の順に押下
3. `Create API Key` を押下し、以下の通り設定の上 `Create & View` を押下
   - `API Key Name` -> 任意（環境毎）
   - `API Key Permissions` -> `Restricted Access`
   - `Access Details`
     - `Email Activity` -> `Read Access`
     - `Inbound Parse` -> `Full Access`
     - `Mail Send` -> `Full Access`
     - `Mail Settings` -> `Full Access`
4. 表示されたキーをメモしておく

- - -

## Sentry
アプリケーションエラー時の通知に使用。

### プロジェクトの作成とDSN(Client Key)の発行
1. 以下のURLにアクセスしログインする
   - https://sentry.io/auth/login/
2. `Project & Teams` -> `New Project` の順に押下
3. 以下のように設定し、 `Create Project` を押下
   - `Choose a language or framework:` -> 選択しない
   - `Give your Project a name:` -> 任意（環境毎）
4. `Get your DSN` を押下
5. 表示された `DSN` と `Public DSN` をメモしておく

- - -

## GitHub
atagoからGitHubへのアクセスに使用する。
### キーペアの生成
キーペアをローカルで生成する

```
# GitHub用のキーペア生成（passphraseは無しとする）
$ ssh-keygen -f ~/.ssh/github_rsa -t rsa -b 4096 -C "<Your mail addresse>"

# 接続設定
$ cat << EOS > ~/.ssh/config
Host github.com
  HostName github.com
  IdentityFile ~/.ssh/github_rsa
  User git
EOS

# 上記の公開鍵をクリップボードにコピーしておく。
$ cat ~/.ssh/github_rsa.pub
```

### GitHubへの公開鍵の登録

1. 以下のURLにアクセス（要GitHubアカウント認証）
   - https://github.com/settings/keys
2. `New SSH key` を押下
3. 以下の通り、情報を入力し `Add SSH key` を押下
   - Title -> `devops`
   - Key -> 上記でコピーした公開鍵をペーストする

### 接続確認

```
$ ssh -T github.com
```

- - -

## Slackの準備
atagoとSlack間の通信に使用する。
### アプリの作成
1. 以下のURLにアクセス（ログイン画面が表示された場合はログインしてください）
   - https://api.slack.com/apps?new_app=1
2. 表示されたポップアップで以下の通り操作
   - `アプリ名` に `atago` を入力
   - `Slackワークスペースの開発` で対象のワークスペースを選択
   - `アプリを作成する` を押下

### アプリのインストール
1. 左側のメニューから `Install App to Workspace` を押下
2. 認証画面に遷移したら `Authorize` を押下
3. `Installed App Settings` の画面に遷移したら `Bot User OAuth Access Token` をコピーしておく

### Bot設定 
1. 左側のメニューから `Bot Users` を押下
2. 以下の通り操作
   - `Add a Bot User` を押下
   - `Display name` と `Default username` に `atago` を入力
   - `Always Show My Bot as Online` を `On` に変更
   - `Add Bot User` を押下

### BotのユーザーID取得
#### Legacy tokenの取得
※ `Legacy` とあるため、将来的に別手段を検討する。
1. 以下のURLにアクセス
   - https://api.slack.com/custom-integrations/legacy-tokens
2. Create tokenボタンを押下してトークンを取得・表示し、クリップボードコピー

#### User IDの確認
1. 以下のURLへアクセス
   - https://slack.com/api/users.list?token=`上記で取得したtoken`
2. JSONが返ってくるので、atagoユーザーを検索し `id` 部分をメモする


### Botのチャンネルへの招待
Slackのチャット画面から、atagoをオペレーション用のチャンネルへ招待する。