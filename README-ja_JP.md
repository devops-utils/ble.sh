[Languages: [:us: English](README.md) | **:jp: 日本語**]
# ble.sh

`ble.sh` (*Bash Line Editor*) はピュア Bash スクリプトで書かれたコマンドラインエディタで、標準の GNU Readline を置き換える形で動作します。
- コマンドラインを (`fish` シェルみたいに) `bash` 構文に従って色付け
- 文法構造に従った補完
- `vim` モードサポートの強化

このスクリプトは `bash-3.0` 以降で利用できます。

現時点では、文字コードとして `UTF-8` のみの対応です。

このスクリプトは [**BSD License**](LICENSE.md) (3条項 BSD ライセンス) の下で提供されます。

> ![ble.sh demo gif](https://github.com/akinomyoga/ble.sh/wiki/images/demo.gif)

## 使い方
**最新の git repository のソースから生成して使う場合**

`ble.sh` を生成する為には `gawk` (GNU awk) と `gmake` (GNU make) が必要です。
以下のコマンドで生成できます。
GNU make が `gmake` という名前でインストールされている場合は、`make` の代わりに `gmake` として下さい。
```console
$ git clone https://github.com/akinomyoga/ble.sh.git
$ cd ble.sh
$ make
```
スクリプトファイル `ble.sh` がサブディレクトリ `ble.sh/out` 内に生成されます。
`source` コマンドを用いて読み込めます:
```console
$ source out/ble.sh
```
指定したディレクトリにインストールするには以下のコマンドを使用します。`INSDIR` の指定を省略したときは既定の場所 `${XDG_DATA_DIR:-$HOME/.local/share}/blesh` にインストールされます。
```console
$ make INSDIR=/path/to/blesh install
```

**`ble.sh` をダウンロードして使う場合** (201512安定版・古いです)

`wget` を使う場合:
```console
$ wget https://github.com/akinomyoga/ble.sh/releases/download/v0.1.8/ble-0.1.8.tar.xz
$ tar xJf ble-0.1.8.tar.xz
$ source ble-0.1.8/ble.sh
```
`curl` を使う場合:
```console
$ curl -LO https://github.com/akinomyoga/ble.sh/releases/download/v0.1.8/ble-0.1.8.tar.xz
$ tar xJf ble-0.1.8.tar.xz
$ source ble-0.1.8/ble.sh
```

指定したディレクトリに `ble.sh` を配置するには単に `ble-0.1.7` ディレクトリをコピーします。
```console
$ cp -r ble-0.1.7 /path/to/blesh
```

**`.bashrc` の設定**

対話シェルで常用する場合には `.bashrc` に設定を行います。以下の様にコードを追加して下さい。
```bash
# bashrc

# .bashrc の先頭近くに以下を追加して下さい。
if [[ $- == *i* ]] && source /path/to/blesh/ble.sh noattach; then
  # ble.sh の設定 (後述) はここに記述します。
fi

# 間に通常の bashrc の内容を既述します。

# .bashrc の末端近くに以下を追加して下さい。
((_ble_bash)) && ble-attach
```

## 基本設定
殆どの設定は `ble.sh` を読み込んだ後に指定します。
```bash
...

if [[ $- == *i* ]]; then
  source /path/to/blesh/ble.sh noattach
  
  # ***** 設定はここに書きます *****
fi

...
```

**Vim モード**

Vim モードについては [Wiki の説明ページ](https://github.com/akinomyoga/ble.sh/wiki/Vi-(Vim)-editing-mode) を御覧ください。

**曖昧文字幅**

設定 `char_width_mode` を用いて、曖昧文字幅を持つ文字 (Unicode 参考特性 `East_Asian_Width` が `A` (Ambiguous) の文字) の幅を制御できます。
現在は 3 つの選択肢 `emacs`, `west`, `east` が用意されています。
設定値 `emacs` を指定した場合、GNU Emacs における既定の文字幅と同じ物を使います。
設定値 `west` を指定した場合、全ての曖昧文字幅を 1 (半角) と解釈します。
設定値 `east` を指定した場合、全ての曖昧文字幅を 2 (全角) と解釈します。
既定値は `east` です。この設定項目は、利用している端末の振る舞いに応じて適切に設定する必要があります。
例えば `west` に設定する場合は以下の様にします:

```bash
bleopt char_width_mode='west'
```

**文字コード**

設定 `input_encoding` は入力の文字コードを制御するのに使います。現在 `UTF-8` と `C` のみに対応しています。
設定値 `C` を指定した場合は、受信したバイト値が直接文字コードであると解釈されます。
既定値は `UTF-8` です。`C` に設定を変更する場合には以下の様にします:

```bash
bleopt input_encoding='C'
```

**ベル**

設定 `edit_abell` と設定 `edit_vbell` は、編集関数 `bell` の振る舞いを制御します。
`edit_abell` が非空白の文字列の場合、音による通知が有効になります (つまり、制御文字の `BEL` (0x07) が `stderr` に出力されます)。
`edit_vbell` が非空白の文字列の場合、画面での通知が有効になります。既定では音による通知が有効で、画面での通知が無効になっています。

設定 `vbell_default_message` は画面での通知で使用するメッセージ文字列を指定します。既定値は `' Wuff, -- Wuff!! '` です。
設定 `vbell_duration` は画面での通知を表示する時間の長さを指定します。単位はミリ秒です。既定値は `2000` です。

例えば、画面での通知は以下のように設定・有効化できます:
```bash
bleopt edit_vbell=1 vbell_default_message=' BEL ' vbell_duration=3000
```

もう一つの例として、音による通知は以下の様にして無効化できます。
```bash
bleopt edit_abell=
```

**着色の設定**

構文に従った着色で使用される、各文法要素の色と属性は `ble-color-setface` シェル関数で設定します。
既定の設定は以下のコードに対応します:
```bash
ble-color-setface region                   bg=60,fg=white
ble-color-setface disabled                 fg=gray
ble-color-setface overwrite_mode           fg=black,bg=51
ble-color-setface syntax_default           none
ble-color-setface syntax_command           fg=red
ble-color-setface syntax_quoted            fg=green
ble-color-setface syntax_quotation         fg=green,bold
ble-color-setface syntax_expr              fg=navy
ble-color-setface syntax_error             bg=203,fg=231
ble-color-setface syntax_varname           fg=202
ble-color-setface syntax_delimiter         bold
ble-color-setface syntax_param_expansion   fg=purple
ble-color-setface syntax_history_expansion bg=94,fg=231
ble-color-setface syntax_function_name     fg=purple
ble-color-setface syntax_comment           fg=gray
ble-color-defface syntax_document          fg=94
ble-color-defface syntax_document_begin    fg=94,bold
ble-color-setface command_builtin_dot      fg=red,bold
ble-color-setface command_builtin          fg=red
ble-color-setface command_alias            fg=teal
ble-color-setface command_function         fg=purple
ble-color-setface command_file             fg=green
ble-color-setface command_keyword          fg=blue
ble-color-setface command_jobs             fg=red
ble-color-setface command_directory        fg=navy,underline
ble-color-setface filename_directory       fg=navy,underline
ble-color-setface filename_link            fg=teal,underline
ble-color-setface filename_executable      fg=green,underline
ble-color-setface filename_other           underline
```

色コードはシェル関数 `ble-color-show` (`ble.sh` 内で定義) で確認できます。
```console
$ ble-color-show
```

**キーバインディング**

キーバインディングはシェル関数 `ble-bind` を使って変更できます。
例えば `C-x h` を入力した時に "Hello, world!" と挿入させたければ以下のようにします。
```bash
ble-bind -f 'C-x h' 'insert-string "Hello, world!"'
```

既存のキーバインディングは以下のコマンドで確認できます。
```console
$ ble-bind -d
```

以下のコマンドでキーバインディングに使える関数一覧を確認できます。
```console
$ ble-bind -L
```

## ヒント

**複数行モード**

コマンドラインに改行が含まれている場合、複数行モード (MULTILINE モード) になります。

`C-v RET` または `C-q RET` とすると改行をコマンドラインの一部として入力できます。
複数行モードでは、`RET` (`C-m`) はコマンドの実行ではなく新しい改行の挿入になります。
複数行モードでは、`C-j` を用いてコマンドを実行して下さい。

`shopt -s cmdhist` が設定されているとき (既定)、もし `RET` (`C-m`) を押した時にコマンドラインが構文的に閉じていなければ、コマンドの実行ではなく改行の挿入を行います。

**Vim モード**

`.bashrc` に `set -o vi` が設定されているとき、または `.inputrc` に `set editing-mode vi` が設定されているとき、vim モードが有効になります。
Vim モードの詳細な設定については [Wiki のページ (英語)](https://github.com/akinomyoga/ble.sh/wiki/Vi-(Vim)-editing-mode) を御覧ください。

## 謝辞

- @cmplstofB さまには vi モードの実装のテストをしていただき、またさまざまの提案を頂きました。