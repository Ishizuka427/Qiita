AWSコンソールでポチポチしてみたのですが
ローカル環境での実行をしてみたいなとおもい、その備忘録です。

環境：
macOS High Sierra 10.13.6
AWS SAM CLI
pip 19.2.2
Python 3.7.2
Docker for macOS

---

##目次
1. AWS SAM CLIをインストールする
2. sam init
3. テストを実行
4. ローカルでLambda関数を呼び出す

---
##1. AWS SAM CLIをインストールする

pip を使用して AWS SAM CLI をインストール

```
$ pip install aws-sam-cli
```
バージョンの確認

```
$ sam --version
SAM CLI, version 0.19.0
```

##2. sam init

sam init コマンドで python3.7のサンプルアプリケーションを作成

```
$ sam init --runtime python3.7
[+] Initializing project structure...


Project generated: ./sam-app


Steps you can take next within the project folder
===================================================
[*] Invoke Function: sam local invoke HelloWorldFunction --event event.json
[*] Start API Gateway locally: sam local start-api


Read sam-app/README.md for further instructions


[*] Project initialization is now complete
```
Readとのこと =>　https://github.com/yoshiyoshifujii/sam-app/blob/master/README.md

ツリーで見てみる

```
$ tree sam-app/
sam-app/
├── README.md
├── event.json
├── hello_world
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-37.pyc
│   │   └── app.cpython-37.pyc
│   ├── app.py
│   └── requirements.txt
├── template.yaml
└── tests
    └── unit
        ├── __init__.py
        ├── __pycache__
        │   ├── __init__.cpython-37.pyc
        │   └── test_handler.cpython-37.pyc
        └── test_handler.py

5 directories, 12 files
```

##3. テストを実行

テストを実行してみる

```
$ cd sam-app
```

```
$ python -m pytest tests/ -v
========================================== test session starts ==========================================
platform darwin -- Python 3.7.2, pytest-5.1.0, py-1.8.0, pluggy-0.12.0 -- /Library/Frameworks/Python.framework/Versions/3.7/bin/python3
cachedir: .pytest_cache
rootdir: /Users/suwa/sam-app
plugins: mock-1.10.4
collected 1 item                                                                                        

tests/unit/test_handler.py::test_lambda_handler PASSED                                            [100%]

=========================================== 1 passed in 0.07s ===========================================
```
ビルドします

```
$ sam build
```

##4. ローカルでLambda関数を呼び出す

まずDockerを起動する
ローカルで API Gateway 経由で Lambda 関数を呼び出す

```
$ sam local start-api
```
ブラウザで下記URLにアクセスするとコンテナイメージがダウンロードされ、コードが実行されてブラウザに出力が表示されます。
http://127.0.0.1:3000/hello
{"message": "hello world”}

他のblogではLocationが表示されていたのですが

```
$ vi ./hello_world/app.py
```
で、下にある

```
# "location": ip.text.replace("\n", "")
```
の、コメントアウトを外せばLocationも表示されます。

