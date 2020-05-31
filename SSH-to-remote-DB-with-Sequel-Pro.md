# はじめに

Sequel Proを使用してWordPressなどのリモートDBへSSH接続する際のメモです。

DBクライアントが内部でどういった動きをしているのかを把握して
実際にCLIで確認しながら設定をします。

# SSHホストの内容を確認

```console
$ ssh ${ユーザー名}@52.196.***.** -i ~/.ssh/hoge.pem
```

もしくは

```console
$ cat ~/.ssh/config 
```

```config:~/.ssh/config
Host hoge-server
  HostName 52.196.***.**
  User ${ユーザー名}
```

```console
$ ssh hoge-server
```

# MySQLホストの内容を確認 (MySQLへログイン)

サーバーへログイン後..

```console
$ mysql -u ${ユーザー名} -h 127.0.0.1 -D ${データベース名} -p
Enter password:🔑
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/7e261f01-ff1b-428f-7cf7-2427daf3f802.png)

接続が成功しました👏

# 解説

## リモートサーバー内のMySQL情報

項目|内容
---|---
MySQLホスト|リモートサーバー内のMySQLホスト名を入力します※
ユーザー名|MySQLのユーザー名を入力します
パスワード|MySQLのパスワードを入力します
データベース|指定する場合はデータベース名を入力します
ポート|デフォルトは3306です

※localhostで指定した場合、TCP/IPを使用した接続ではなくソケットファイルでの接続になり
ポート指定のオプションが無視されるため、基本的には`127.0.0.1`で接続をします。

## リモートサーバーに接続するための情報

項目|内容
---|---
SSHホスト|ssh接続する先(リモートサーバー)のホスト名を入力します
SSHユーザー|ssh接続する先のユーザー名を入力します
SSHパスワード|パスワードを設定している場合は入力します※
SSHポート|ポートの指定がある場合は入力します

※`~/.ssh`配下にある鍵は自動的にcheckされます。

# 切り分けて考える

DBクライアントが内部でどういった動きをしているのかが分かると
例えば接続がうまくいかない際に
「そもそもリモートサーバーへのssh接続はできているのか？」
など、問題を切り分けて対処することができます。

SSHホストとSSHパスワード
MySQLのホスト名のみを入力して接続を試みてみます。
(MySQLのユーザー名とパスワードは、あえて入力しません)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/7a5b4257-714a-94c1-83b3-697df46587cf.png)

この状態で接続をすると

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/b4a518b3-34b3-173d-08dd-357f450de218.png)

このように、エラー文に『MySQLの応答』が表示されるため
**リモートサーバーへのssh接続には成功している**
と、判断することが出来ます。



