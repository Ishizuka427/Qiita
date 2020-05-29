# 環境
macOS

# AWS CLI インストール

```
brew install awscli --upgrade --user
```

`hostname doesn't match ` というエラーが出た場合は？
=> 確認: Python のバージョンは 2.7.9 以上か

# アクセスキーの作成

AWS CLIには
使用している環境でAWS認証情報が設定されている必要がある。

AWSの認証情報を設定するには、使用するIAMユーザーにてアクセスキーを作成する必要がある。
ダッシュボードのここ。

<img width="1117" alt="スクリーンショット 2020-05-22 10.35.58.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/70653c90-6bfd-b5f6-b8aa-1af8c068ed96.png">


`AWS Access Key ID`と`AWS Secret Access Key`が発行されるので保存する。

# コマンド一発設定(既に設定してある場合は上書きされる)

`$ aws configure`を実行すると
AWS CLI によって`アクセスキー`、`シークレットアクセスキー`、`AWSリージョン`、`出力形式`、以上4つの情報の入力が求められる。これらは自動的に`default`という名前のプロファイル設定群に保存される。(既に設定してある場合は上書きされる)

# 手動設定(複数プロファイル管理)

- `~/.aws`以下のconfigとcredentialsファイルで確認可能。
- 手動修正するなどして、複数プロファイル管理も可能。

```
$ aws configure
AWS Access Key ID [None]: 🆔＊＊＊＊＊
AWS Secret Access Key [None]: 🔑＊＊＊＊＊
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

# AWS認証プロファイルの設定

AWS認証プロファイルの設定を効かせます。

```
$ export AWS_PROFILE=don.suwa3.me
```

現在のセッションにのみ適用されます。

```
$ env
AWS_PROFILE=don.suwa3.me
```

わたしはセッションごとに忘れずexportする運用をしています
理由
- 仕事用と個人用でAWS CLIを複数アカウント運用しているため(本当はダメかも)
- 環境変数を`bash_profile`に固定してしまうと、あとでそのことを忘れそう

ツールでディレクトリごとAWS認証プロファイル設定ができるものもあるらしいです。


