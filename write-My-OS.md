# はじめに

かねがねOSの自作をしてみたいなと思っていたので
書籍などを参考に、OS自作をガッツリ始める前に
まずは簡易的なOSの実装にチャレンジしてみました。

ちょうど手頃なチュートリアルを見つけたので
試してみたものを、簡単な解説とともに作業ログ的にまとめました。

実際に作成したコードはこちらになります
https://github.com/Ishizuka427/MyOS

# 環境
macOS

# 参照

主に参考にした記事＆書籍です

- [Babystep1 - OSDev Wiki](https://wiki.osdev.org/Babystep1)
- [OSを書く：初歩から一歩ずつ | POSTD](https://postd.cc/writing-an-os-baby-steps/)
- [０から作るOS開発　Vol.1　ブートローダー編 Kindle版](https://www.amazon.co.jp/%EF%BC%90%E3%81%8B%E3%82%89%E4%BD%9C%E3%82%8BOS%E9%96%8B%E7%99%BA-Vol-1-%E3%83%96%E3%83%BC%E3%83%88%E3%83%AD%E3%83%BC%E3%83%80%E3%83%BC%E7%B7%A8-%EF%BC%90%E3%81%8B%E3%82%89%E4%BD%9C%E3%82%8B%E3%83%96%E3%83%BC%E3%83%88%E3%83%AD%E3%83%BC%E3%83%80%E3%83%BC-yabusame2001-ebook/dp/B00F6CFO9O)

# 実行環境を整える

まず、実行環境ですが
ホストマシンに

- nasm
- make
- qemu

をインストールします。

※DockerやVMなどを用いてローカル環境を汚さずに行うことも可能です。
ただ今回の内容程度であれば、特に問題ないと判断しました。
今後、自作OSを続けたい場合はgit管理などで仮想マシンへの移動をスムーズにすると良いかもしれません。

```
$ brew install nasm
$ brew install make
$ brew install qemu
```

## 🐾解説

`nasm`
アセンブラです。
機械が理解できる命令プログラム**アセンブリコード**を
機械が理解できる命令**バイナリコード**に置き換えて、コンピューターが実行できるようにします。

`make`
コンパイルを自動化するツールです。

`qemu`
キューエミューと読みます。
自作したOSは、このエミュレーターを介して実行されます。
いわゆる仮想マシンです。

# コンピューターの起動について

コンピューターが起動してからの処理について
わかりやすい概要を見つけたので記載します。

>電源ボタンをONするとまずBIOS（BasicInputOutputSystem）が動き出します。BIOSはその名の通り基本的な（Basic）入力（Input）と出力（Output）を制御するハードウェアとソフトウェアのシステム（System）です。（このあたりは別に読み飛ばしても問題ありません）。BIOSが動き出すとPOST（PowerOnSelfTest）と呼ばれている処理を行います。POST処理では接続されたデバイスのチェック・初期設定、メモリーのチェックを行って正常にシステムが起動できるかどうかをチェックします。起動できると判断すると次にブートローダーまたはOSをメモリーにロードします。ブートローダーまたはプログラムがロードされた後にそのプログラムが様々な処理を簡単に行えるように、BIOSは入出力デバイスの操作をプログラムから利用できるインターフェースを用意しています。
--「0から作るOS開発」から引用--

MBR(マスターブートレコード)の容量には上限があるため、それ以上のロードにはブートローダーが用いられます。
今回作成するOSはとてもシンプルなため、ブートローダを使用してさらにコードをロードすることはしません。

# アセンブリコードを書いてみる

実際にアセンブリコードを書いてみます。

```
$ vi boot.asm
```

```nasm:boot.asm
; boot.asm
hang:
jmp hang
 
times 510-($-$$) db 0
 
; This is a comment
 
db 0x55
db 0xAA
```

## 🐾解説

`hang`
コード内の名前つきマーカーです。

`jmp hang`
マーカーhangにジャンプします。これで無限ループになります。

`times 510-($-$$) db 0`
残りのバイトを0で埋めるためのnasm構文です。

`$`
現在の行の先頭

`$$`
ファイルやセクションの先頭

`($-$$)`
ファイルの先頭から現在の位置を引く
2つのdbコマンドが、最後に2バイト分で格納するので
MBRの残りの512バイトを埋めるのに512という数を使いません。

`;`
コロン以降はコメントアウトになります。

`0x55と0xAA`
BIOSに「はい、これは実行可能なMBRです」と知らせます。
作成したファイルをバイナリファイルに変換して、コンピューターが実行できるようにします。

```console
$ nasm -f bin boot.asm -o boot.bin
```

## 🐾解説

`-f bin`
nasmがバイナリフォーマットにアセンブルします。

`boot.asm`
nasmがアセンブルしようとしているアセンブリファイルです。

`boot.bin`
出力ファイル名です。

エミュレーターで実行してみます。

```
$ qemu-system-x86_64 boot.bin
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/f583ea74-4001-e6ad-913c-9d87ddd91d06.png)

アセンブリファイルをコンパイルして実行することができました👏

# Makefileの作成

次に、Makefileを作成してコンパイルを自動化します。

```
$ vi Makefile
```

```make:Makefile
boot.bin: boot.asm
	nasm -f bin boot.asm -o boot.bin
 
qemu: boot.bin
	qemu-system-x86_64 boot.bin
 
clean:
	rm *.bin
```

## 🐾Makefileの文法解説

```makefile:Makefile
ターゲット名α: 依存ファイル名α
    コマンド行α

ターゲット名β: 依存ファイル名β
    コマンド行β

ターゲット名γ: 依存ファイル名γ
    コマンド行γ
```

ターゲット名を指定してmakeを実行する場合は
`$ make ターゲット名`
とします。また、コロンの後の値は依存ファイルです。

例①

```
$ make qemu
```
👆こちらを実行すると、**make**は`boot.bin`のコマンド（`nasm -f bin boot.asm -o boot.bin`）を実行したあと、`qemu-system-x86_64 boot.bin`を実行します。

例②

```
$ make clean
```

👆こちらを実行すると、`rm *.bin`が実行され、すべてのアセンブルされたファイルが削除されます。

※ターゲットを省略するとMakefileの中で先頭のターゲットが実行されます。
**[注意⚠]Makefile内でコマンド行を記述する場合は、必ず先頭にタブ文字を入れてください。**

# 文字を表示してみる

`boot.asm`アセンブリファイルを書き換えて、スクリーンに文字を表示させてみます。

```console
$ vi boot.asm
```

```nasm:boot.asm
; boot.asm
mov ax, 0x07c0
mov ds, ax
 
mov ah, 0x0
mov al, 0x3
int 0x10
 
mov si, msg
mov ah, 0x0E
 
print_character_loop:
lodsb
 
or al, al
jz hang
 
int 0x10
 
jmp print_character_loop
 
msg:
db 'Hello, World!', 13, 10, 0
 
hang:
jmp hang
 
times 510-($-$$) db 0
 
db 0x55
db 0xAA
```

ファイルを編集したら実行してみます。

```
$ make clean
$ make qemu
```

## 🐾解説

```
mov ax, 0x07c0
mov ds, ax
```

`mov`はデータを動かす命令です。
ここでは`0x07c0`を汎用レジスタ`ax` (アキュムレータ)へ移動したあとに`ax`を`ds`(データセグメント)にロードします。
BIOSは毎回`0x07c0:0x0000`の位置にMBRをロードするため、このように設定します。
`ds`はレジスタから移動してきたデータのみを受け取る仕様のため、このように段階を踏んでデータの受け渡しを行います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/9c0f9b50-14d5-afb3-434e-befb86028db8.png)


`レジスタ`
CPU内の記憶装置です。
それぞれに役割があり、例えば今回のケースではセグメントレジスタに値を入れることでセグメントの開始位置が変わります。
0x07C0をdsレジスターに格納することで、セグメントの開始位置が0x7C00になります。

`セグメント`
メモリーをセグメントと呼ばれるある程度の大きさに区切ったものをいいます。
全容量の上限は決まっていますが、一つ一つの大きさやメモリー上の場所を自分で変更することが可能です。
また、セグメント同士の領域はお互い重複するようにすることもできます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/424a129f-0636-5fb3-a33d-e9a7a2098907.png)

(参照: 「0から作るOS開発」)

ファイルの中身についてざっくりと解説をします。

```nasm:(解説用)
; boot.asm
mov ax, 0x07c0    ; 0x07c0 を ax に動かす
mov ds, ax        ; ax を dx に動かす

mov ah, 0x0       ; 0x0 を ah に動かす
mov al, 0x3       ; 0x3 を al に動かす
int 0x10          ; 上記を 0x10 (ビデオサービスを管理するBIOS)に int命令で割り込み 
 
mov si, msg       ; msg を si へ動かす。ソースインデックスに msg を格納
mov ah, 0x0E      ; 0x0E を ah へ動かす。画面の文字列を 0x10 へ割り込み可能に

; ループ開始
print_character_loop:    
lodsb             ; lodsb命令 si から al に1byteロードする  
 
or al, al         ; al(0x3)と(0x3)をor演算。文字列が0になり終わると、プロセッサはオペランドの位置にジャンプ 
jz hang

int 0x10          ; 文字列を画面にプリントする

jmp print_character_loop    ; ループの先頭に戻り、文字列が終わりでなければ次の文字をプリントする。
; ループここまで

msg:
db 'Hello, World!', 13, 10, 0    ; db => これらの値をmsg:アドレスに格納するよう命じる

hang:
jmp hang

times 510-($-$$) db 0

db 0x55
db 0xAA
```

Hello, Worldが表示された画面がこちらです👏

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/449552/f9a20795-e94b-dd3c-5ede-b7ab2fdb7e1e.png)

# Nextステップにオススメの書籍

[30日でできる! OS自作入門 Kindle版](https://www.amazon.co.jp/dp/B00IR1HYI0/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1)



