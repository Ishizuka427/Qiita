# 環境
macOS

# Dockerfileの作成

[Django Girlsチュートリアル](https://tutorial.djangogirls.org/ja/)でつくったもの

```console
$ ls 
blog            manage.py        myvenv 
README.md        db.sqlite3        mysite            requirements.txt
```

[Docker Hub](https://hub.docker.com/)で使いたいイメージを探します。
公式のものなど、なるべく信頼できるイメージをつかいます。

元になるイメージはpython:3.7-slim-busterを選びました。
Django起動時に打ち込んでいるコマンドを
Dockerfileに書き込んでいきます。

```
$ vi Dockerfile 
```

```Dockerfile
FROM python:3.7-slim-buster 
COPY . /app/ 
WORKDIR /app 
RUN pip3 install -r requirements.txt 
RUN python3 manage.py collectstatic --noinput 
CMD python3 manage.py runserver 0.0.0.0:8000
```

# Dockerの構築

任意でイメージ名やタグなどの指定をして
buildしてみます。

```console
$ docker build -t [イメージ名]:[タグ] . 
例えば 
$ docker build -t djangogirls:0.1.0 .
```

イメージの一覧を表示させます。

```console
$ docker images 
REPOSITORY        TAG          IMAGE ID             CREATED              SIZE 
djangogirls              latest        53a38ee83c1a      24 minutes ago 
```

## ローカルで何か走っていないか確認👀

**Dockerを走らせる前に**
心当たりがある場合は、ローカルで何か走っていないか確認します。
※特にローカルで走らせているものがなければ、以下読み飛ばしてもOKです。

```console
$ ps -ef | grep runserver 
  502  2437   479   0 土02PM ttys000    0:00.60 /Library/Frameworks/Python.framework/Versions/3.7/Resources/Python.app/Contents/MacOS/Python manage.py runserver 
  502  2438  2437   0 土02PM ttys000   58:01.79 /Library/Frameworks/Python.framework/Versions/3.7/Resources/Python.app/Contents/MacOS/Python manage.py runserver 
  502 26677 13920   0  3:55PM ttys003    0:00.01 grep runserver
```

大まかに書くと

```
2437（子:Django）    479（親:shell） 
2438（子:Python）　  2437（親:Django） 
26677 13920 => いま検索したgrepコマンドが走っているので無視
```

shellは殺さず、2437を殺せば親のDjangoもろとも子の2438も死ぬので
ローカルで走りっぱなしでいたDjangoのrunserverをkillします。

```
$ kill 2437
```

killできたか確認。

```console
$ ps -ef | grep runserver 
  502 26680 13920   0  3:55PM ttys003    0:00.01 grep runserver
```

# Docker run

Dockerを走らせます。
※`djangogirls`は`Docker ID`でもOK。

```console
$ docker run -it --rm -p djangogirls
```

下記にアクセス

http://127.0.0.1:8000/

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/665612d9-d275-27b4-3b2a-594636f37f9e.png)

できた👏

# Tips (.dockerignore)
全体的に重かったので.dockerignoreを作成し、無視するファイルを指定します。

```
$ vi .dockerignore 
```

```config:.dockerignore
myvenv 
db.sqlite3
```

`myvenv`は仮想環境なので不要です。
`db.sqlite3`はDockerに乗せるには今後重くなる可能性が高いので
もしデプロイする際にはMySQLなどを使っても良いかもです。
ちなみにDBをignoreで無視するとDockerを走らせた際に赤文字で

<font color="#ff0000">
You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, blog, contenttypes, sessions. 
Run 'python manage.py migrate' to apply them.
</font>

このようにDBに問題ありなどと表示されます。
あえてignoreしている場合はスルーです。

