# radio-gogaku-downloader

## 説明
NHKラジオ語学講座のストリーミングファイルをダウンロードするためのツールです。

GUI不要で、コマンドラインのみで動作しますので、Linuxサーバー等でcronで定期実行させられます。

講座一覧はJSONファイルの形 (courses-all.json) でツール本体とは分離しています。

ファイル名等で日本語を取り扱う関係で、Linuxの場合はencodingをUTF-8にする必要があります。（注意事項参照）
音声は、64kbpsのMP3に変換されます。

◆◆◆ V2.1以降が2021年度前期の番組対応となります。V2.1は、3月放送分が2020年度のフォルダーに入ってしまいます。
V2.1.1で修正しました。すでに落とした場合はファイルの移動をお願いします…。

◆◆◆ V2.1で、courses-selected.jsonのフォーマットを変更しました。
それ以前のバージョンですでに作成している場合は、一旦このファイルを削除の上、-sオプションで再度作成する必要があります。

## 実行環境
Python 3.xの走る環境。 (3.7以上で動作確認済み)
Linux（含むWSL。たぶんMacOSも可）。
(NEW) Windowsでも-sオプション以外は動作します。（詳細は後述）

#### その他必要なプログラム（あらかじめインストールしておいてください）
- python3-pip
- ffmpeg

#### 必要なPythonパッケージ (あらかじめpip3 installでインストールしておいてください)
- requests
- npyscreen
- ffmeg-python

## 使い方
コマンドラインから実行。

    python3 radio-gogaku-downloader.py [-h] [-s] [-y] [-d DIR] [-o OUTPUT]

### オプション

#### -h, --help
ヘルプの表示

#### -s, --select
講座選択画面の表示。
最初の起動時などcourses-selected.jsonが存在しないときには、このオプションを選択しなくても講座選択画面に進みます。
必要な講座にチェックを入れ、OKを押すと、プログラムと同じディレクトリにcourses-selected.jsonというファイルを作ります。

#### -d DIR, --dir DIR
音声ファイルを保存する際のルートディレクトリの指定。
音声ファイルはここで示したディレクトリの下のサブディレクトリに格納されます。
-h, -s を指定したとき以外は必須です。音声をダウンロードする場合は、必ず指定してください。

#### -y YEAR, --year YEAR
ファイル名の放送日に年を加えるかの指定。0:年表示なし 1:年表示あり。デフォルトは1(年表示あり)。
ver 1 同様に月日のみのファイル名にする場合は 0 を指定。

#### -o OUTPUT, --output OUTPUT
保存ディレクトリ名、ファイル名のオプション。OUTPUTには、以下の1～4のいずれかを指定してください。このオプションがない場合は、1を指定したものとみなします。

1. 指定したディレクトリ/年度/講座名/日付.mp3
1. 指定したディレクトリ/年度/講座名/講座名_日付.mp3
1. 指定したディレクトリ/年度/講座名_日付.mp3
1. 指定したディレクトリ/講座名/年度/日付.mp3

#### -f, --force
強制上書き。デフォルトでは、すでに同名のファイルが存在した場合は上書きしません。
このオプションを付けると、既存の同名のファイルがあった場合に上書きします。

## 注意事項
### その1
使用しているモジュールの関係でWindowsネイティブ環境（PowerShell, コマンドプロンプト）では、-sオプションは動きません。（WSL上でなら動作します）
npyscreenが、Unixのcursesというライブラリを使っているのがその理由です。（windows-cursesは日本語対応していないようです）

Windowsネイティブ環境で使う方法は、「その4」を参照してください。

### その2
Linux環境で、下記のようなエラー（ロケールや文字コードに関係ありそうなエラー）が出る場合、encodingがUTF-8になっていません。
（このツールの実行には、encodingをUTF-8にする必要があります。 Linuxのみ。Windows (WSL)やMac OSでは出ないはず）

- UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position 147: ordinal not in range(128)
- locale.Error: unsupported locale setting

```
$ locale
```
と打ってみて、出力された中でLANGに"UTF-8"の文字が含まれていればエラーは出ないはずです。
設定例として、例えば、~/.bashrcの最後に以下を追加するとか。
要はencodingがUTF-8になっていればやり方は問いませんので、お好みのやり方で。
```
export LANG=C.UTF-8
export LANGUAGE=en_US:en   <-- エラーメッセージ等日本語にしたい人は不要
```

### その3
ストリーミングのファイルは毎週月曜日に前の週の内容に更新され、その際にそれ以前のファイルは消されます。
ただ、ボキャブライダーだけは、過去のファイルは削除されずに追加だけされています。
そのため、初回および強制上書きオンにすると非常に多くのファイル（2017年度以降すべて）がダウンロードされます。
これはサイト側の仕様なのでご容赦ください。

### その4
Windowsネイティブ環境での使用について。
- PowerShellもしくはコマンドプロンプトから動かせます。バッチファイルとそのショートカットを作って適当なところに置いておけば、
  ダブルクリックだけで実行できて便利かと思います。
- 事前にPython 3およびライブラリ等をインストールする必要があります。（他環境の場合と同じ）
- ffmpeg.exeをパスの通ったところ、もしくは作業フォルダ (radio-gogaku-downloader.pyと同じフォルダ) に置きます。
  - https://www.gyan.dev/ffmpeg/builds/packages/ にある、
    ffmpeg-4.3.2-2021-02-27-full_build.7z で動作確認しています。
- ライブラリの関係で、PowerShellやコマンドプロンプト上では
  -s オプションでメニューから番組を選択することができません。
  代替手段として、次の方法で番組リストを手動で作ってください。
  1. courses-all.jsonを、courses-selected.jsonの名前でコピーする。
  1. courses-selected.jsonを、メモ帳もしくはUTF-8の扱える（開いてみて文字化けしない）テキストエディタで開く。
  1. 番組一覧から、不要な番組を含む行を行単位で削除。（不要な番組以外の行は全て残すこと）
  1. ファイルを保存。
  1. コマンドラインから実行。なおパスの区切り文字は / と \ のいずれでも可。

## 履歴
### V2.2.0
- Windowsの限定サポート。courses-xxxx.jsonをマニュアルで編集しやすいように少し変更。

### V2.1.1
- 新講座が3月中に放送開始の場合、3月放送分が前の年度に扱われてしまう問題への対応。

### V2.1
- 2021年度前期の講座が正式に決まったので、番組一覧を訂正。
- プログラムを更新、course-all.jsonの中に連番で入れていた数字を廃止。

### V2.0.1
- 番組一覧を2021年度前期の講座内容に合わせて更新。

### V2.0
- 2021年度から元サイトの情報管理がXMLベースからJSONベースに変更になるので、それに合わせた変更。
- 「年度」情報はcourses-all.jsonからでなく、サイトのJSONから取るように変更。（放送する講座に変更がない限り年度頭でcourses-all.jsonの更新の必要なし）
- ファイル名が放送日の「月+日」だったのを「年+月+日」に変更。（オプション -y でv1同様年抜きにするのも可能）
- 出力のディレクトリ構成のバリエーション追加。

### V1.0
- 最初のバージョン。
## TODO
- 現時点で特になし。

## その他
ミスで、過去のスターなど消してしまいました…。:cry:
