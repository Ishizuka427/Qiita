https://qiita.com/suwa3/items/0a180e9833c0146e36b5
# ShellコマンドでGitHubへソースコードの反映

Atomから直接GitHubへリポジトリの反映を試みましたが
うまくいかずよくわからなかったので
ターミナルからShellで反映させる方法をまとめました。
環境：macOS

---

## 目次
1. GitHubにリポジトリを作る
2. GitHubからソースコードをDLする
3. Gitディレクトリへ移動
4. Gitへ反映させるFileの移動など
  - おまけ:gitディレクトリ内の整理
5. GitHubにあげてみる
6. GitHubに反映させる
  - おまけ:ステータスの確認
  - おまけ:logの確認

---
## 1. GitHubにリポジトリを作る

作成したリポジトリ画面からSSHのURLをコピーする。
![スクリーンショット 2019-07-06 14.32.37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/30c0101f-84b5-c7d3-7cb2-c47b2378a4a9.png)
![スクリーンショット 2019-07-06 13.34.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/52325ad3-5258-b240-463b-ff86f12a4cd3.png)

## ２. GitHubからソースコードをDLする

```
$ git clone "[URL]"
```
例えば画像のURLを例にすると、実際にはこうなる。

```
$ git clone "git@github.com:suwa3/suwa3-home.git"
```
<br>
念のためlsでディレクトリを確認します。

```
$ ls
```
![スクリーンショット 2019-07-06 13.41.56.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/3d08d15c-48fe-f572-a366-ba83d3c20fff.png)

## 3. gitディレクトリへ移動

```
$ cd suwa3-home/
```
ドットがつくと隠しファイルになってしまうので
-a(allという意味)で表示させる。

```
$ ls -a
```
![スクリーンショット 2019-07-06 15.09.36.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/0d65ca0b-4ba9-cf71-858b-16e1c0588bc7.png)

Gitディレクトリ内の.(ドット)=カレントを開きます。

```
$ open .
```
## 4. Gitへ反映させるFileの移動など

Finderを開き、GitHubにUPしたいFileをコピペなり移動しておくなりする。
![スクリーンショット 2019-07-06 13.54.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/3a08bf43-cb4f-5ff0-b39a-652703ff2bf2.png)

今回はAtomを使用しているので
Fileがどこにあるのかわからない場合は、Projectから場所を確認。
![スクリーンショット 2019-07-06 14.42.34.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/f7fd772d-d1b9-a7f0-1c95-0649009afb47.png)

```:おまけ（gitディレクトリ内の整理）
必要に応じてgitのディレクトリ内を整理する。
今回はMyPortfolioの下階層にHTMLやCSSファイルがあったため、それらを上の階層に移動させました。
$ mv MyPortfolio/[File名] .

一つずつ移動させる例
$ mv MyPortfolio/index.html .

今回のように、複数ファイルがある場合
$ mv MyPortfolio/* .
アスタリスクは全てという意味。
```
## 5. GitHubにあげてみる

ステージングエリアに上げ、登録対象にする。

```
$ git add [ファイルパス]
```

```:例
＄　git add MyPortfolio/index.html
```

<br>
ローカルのリポジトリに登録する。

```
$ git commit
```
vimでコミットメッセージを編集する。
どういった内容か、概要を書く。

## 6. GitHubに反映させる

```
$ git push
```

GitHubでリポジトリ内に反映されているかを確認。
![スクリーンショット 2019-07-06 14.31.59.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/95856ad3-54eb-faf7-e26b-984647ef3bac.png)
わーい:smiley::clap:
<br>

```shell:おまけ(ステータスの確認)
$ git status
```
add以降であれば確認可能。
![スクリーンショット 2019-07-06 14.16.29.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/45e51e32-3e24-c639-2ae7-57852e262981.png)

```shell:おまけ（logの確認）
$ git log
```
コミットできたかlogで確認。
commit以降であれば確認可能。
![スクリーンショット 2019-07-06 14.26.53.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/13a3a52b-9c98-e350-085e-721a037e1771.png)

