# Mysql

## 概要

こちらのconfigurationファイルはMySqlを1-master 2-slave replicationの形で作成する。


#### ファイル構成

```bash
mysql
.
├── cm-mysql.yaml
├── do.sh
├── mysql.yaml
├── secrets
│   ├── repl.password
│   ├── root.password
│   └── user.password
└── svc-mysql.yaml
```

> 参照用

`cm-mysql.yaml`ファイルはmasterとslaveのconfigurationを含む`ConfigMap`である。

`svc-mysql.yaml`は`mysql master` と`mysql replicas`のためのサービスを作成する。

`mysql.yaml` はmysqlのmasterとreplicasのinstancesを作成する。masterの方は最初のstatefulSetになり（indexは`0`）、名前が`mysql-0`のpodになる。他のpod(`mysql-1`や`mysql-2`等）はreplicasになる。


## セットアップ

deployments directoryに移動する

```bash
$ cd ~/go/src/hexalink-k8s/middlewares/mysql
```

### Passwords作成

パスワードの修正は手動修正と自動修正二つの方法がある。

---
#### 手動修正：

パスワードを下記のコマンドでファイルを開き、その内容を修正することで手動修正する。 

```bash
$ vi secrets/user.password  # password for `devops` user
（このコマンドを実行後ファイルを開く。`i`を入力し編集モードに入る。編集完了すれば`esc`で参照モードに戻る。その後、`:wq`で保存してファイルを閉じる）

$ vi secrets/root.password  # password for the `root` user
（このコマンドを実行後ファイルを開く。`i`を入力し編集モードに入る。編集完了すれば`esc`で参照モードに戻る。その後、`:wq`で保存してファイルを閉じる）

$ vi secrets/repl.password  # password for the `replication` user
（このコマンドを実行後ファイルを開く。`i`を入力し編集モードに入る。編集完了すれば`esc`で参照モードに戻る。その後、`:wq`で保存してファイルを閉じる）

```
---
#### 自動修正

下記のコマンドでパスワードを自動修正する

```bash
$ ./do.sh create_new_passwords
```

>このコマンドを実行することで`secrets/` ディレクトリにパスワードを記載しているファイルを作成する。必要な場合、そのファイルを切り取りしてください。

下記のログが表示すれば良い

```bash
Done!

Created secrets/user.password
Created secrets/root.password
Created secrets/repl.password
```

##### 補足：

`$ ./do.sh create_new_passwords`実行時に
`Password files exist. Please move them first.`を表示すれば、下記のコマンドでファイル内容を確認する。

```
$ cat secrets/repl.password
$ cat secrets/root.password 
$ cat secrets/user.password
```
全てのファイル内容が`changeme`であれば、下記のコマンドを実行してから`$ ./do.sh create_new_passwords`をもう一度実行してください。

```
$ rm secrets/repl.password
$ rm secrets/root.password 
$ rm secrets/user.password
```
ファイル内容が`changeme`ではなければ、下記のコマンドを実行してから`$ ./do.sh create_new_passwords`をもう一度実行してください。

```
# ファイルバックアップ作成
$ cp secrets/repl.password secrets/repl.password.bk.`date "+%Y%m%d_%I%M%S"`
$ cp secrets/root.password secrets/root.password.bk.`date "+%Y%m%d_%I%M%S"`
$ cp secrets/user.password secrets/user.password.bk.`date "+%Y%m%d_%I%M%S"`

#ファイル削除
$ rm secrets/repl.password 
$ rm secrets/root.password 
$ rm secrets/user.password 
```


---
#### Secretを作成する

```bash
$ ./do.sh create_secret
```

### Node-Poolを選択

デフォルトでpodが`pool-db0` node-poolにアサインされる。
> 該当のnode-poolがなければエラーになる

**別のnode-poolにアサインしたい場合、こちらの手順を参考してください: [Node-Poolを選択](selecting_node-pool.md).**

## Deployment

下記のコマンドはディレクトリにある全ての`yaml`ファイルを`kubectl create`する。

```bash
$ ./do.sh deploy --all-yaml
```

下記のコマンドでcomponentを確認する。podの起動が少し時間がかかるので、3分ほどお待ちください。

```bash
$ kubectl get pods -l component=mysql
# 全podのSTATUS列が「running」で、READY列に分母と分子が一致していることを確認
```

---

> ##### Future Notes:
>（翻訳保留中）
> - Other microservices/middlewares should instead use the secret instead of copy/pasting it to their config files
