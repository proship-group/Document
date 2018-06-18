# 説明

## 書き方説明

本手順の書き方についてを説明する。

1. `<>` に囲まれる変数は、必須な引数。しかも値の入れ替えが必要。

2. `[]` に囲まれた変数は、省略可能な引数

- 例1

```
$ ssh -i ~/.ssh/devops_rsa devops@<IPアドレス>
```

下記のコマンドは実行可能

```
$ ssh -i ~/.ssh/devops_rsa devops@123.123.123
```

下記のコマンドは実行不可

```
$ ssh -i ~/.ssh/devops_rsa devops@
```

- 例2

```
$ ./do.sh change_password [新しいパスワード]
```

下記のコマンドは実行可能

```
$ ./do.sh change_password 
$ ./do.sh change_password ABC
```