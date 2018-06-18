# `devops`VMへの接続設定
## 前提条件
- ローカル環境がMac

## 秘密鍵設置

`devops`VM接続用の秘密鍵を取得する。（秘密鍵の連携依頼は開発者に連絡してください。）

下記のコマンドを実行してください。

```
$ vi ~/.ssh/devops_rsa
# 「i」入力し、編集モードに入る。取得した秘密鍵をペーストする。編集終了後「esc」ボタンを押して閲覧モードに戻る。「:wq」を入力し保存してviを閉じる。
```

## `devops`VMのIP取得

`GCPコンソール`-`pit-shared-prod`プロジェクト-`Compute Engine`-`VMインスタンス`で、`devops`の***外部IP***をメモする。

## `devops`VMにSSHログイン

下記のコマンドを実行してください。

```
$ ssh -i ~/.ssh/devops_rsa devops@<上記で取得したIPアドレス>
```
下記のメッセージを表示すれば、アクセスできた。

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

