Crostini を入れて使えるようになるまで

100.0.4896.133 時点

## Chrome OS の設定

### 画面ロックの設定

- PIN を設定する
- Smart Lock を設定する (「デバイスのロック解除のみ」)

### 日本語入力の設定

- 辞書とかスペースとか

### ネットワークの設定

- Wi-Fi
- VPN

## Crostini の設定

### Crostini を有効にする

- 設定アプリから

### 日本語入力をできるようにする

```
$ sudo apt install fcitx fcitx-mozc

# 自動起動
$ echo "/usr/bin/fcitx-autostart" >> ~/.sommelierrc

# IME 切り替えキーの設定 / Mozc の選択
$ fcitx-configtool

# 辞書とかスペースとかの設定
$ /usr/lib/mozc/mozc_tool --mode=config_dialog
```

/etc/systemd/user/cros-garcon.service.d/cros-garcon-override.conf に追記する。

```
Environment="GTK_IM_MODULE=fcitx"
Environment="QT_IM_MODULE=fcitx"
Environment="XMODIFIERS=@im=fcitx"
```

fcitx-configtool で Mozc を追加するとき、`Only show current language` のチェックに注意


### konsole のインストール (追記あり)

日本語入力とクリップボードがどちらも動いたので、konsole にした。Terminator も問題なく動作しそうなので、好みで選ぶ。

gnome-terminal は日本語入力が正しく動かず、xterm は起動が速くて好みだったがクリップボードが動作せず。

```
$ sudo apt install konsole
```

### (追記) xterm のインストール

クリップボードを動作させることができた。他のターミナルよりも圧倒的に起動が速い。

```
$ sudo apt install xterm
$ echo "XTerm*selectToClipboard: true" >> ~/.Xdefaults

# .xsessionrc などは実行されなかったので仕方ない
$ echo "xrdb ~/.Xdefaults" >> ~/.bashrc
```

### MTU の設定 (VPN 接続時)

Chrome OS 側で VPN を設定していると、Crostini からネットワークにつながったり繋がらなくなったりする。必要に応じて MTU を小さくする。

```
$ echo "sudo ip link eth0 set mtu <MTU>" >> ~/.sommelierrc
```

### 普段使ってるアプリケーションを入れる

- VSCode
- LibreOffice (なんだかんだあると便利)

## 参考

主に日本語入力周りで参考にした文献。

- [Fcitx5 - ArchWiki](https://wiki.archlinux.jp/index.php/Fcitx5)
- [ja/I18n/Fcitx5 - Debian Wiki](https://wiki.debian.org/ja/I18n/Fcitx5)
- [Chromebook Crostini の有効化と日本語化](https://zenn.dev/igac/articles/84d1f377bcd9d698ee8d)

VPN 設定時の MTU について。

- [VPN and Linux help : Crostini](https://www.reddit.com/r/Crostini/comments/rajbas/vpn_and_linux_help/)

gnome-terminal で日本語入力ができない問題について。

- [Twitter](https://twitter.com/tricken/status/1204380733776596998)
- [Crostiniのfcitxで１文字ずつ勝手に確定される問題の解決方法（暫定）](https://qiita.com/niwashi/items/dc325c44dcc538245cc4)

Xterm について。

- [Xterm - ArchWiki](https://wiki.archlinux.jp/index.php/Xterm)
- [xinit - ArchWiki](https://wiki.archlinux.jp/index.php/Xinit)
