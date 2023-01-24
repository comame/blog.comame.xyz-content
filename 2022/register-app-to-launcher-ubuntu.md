.appimage 形式の実行ファイルをインストールするとき、手動でランチャーに登録しないとならないようだったので、その手順をメモ。

`~/.local/share/applications/<app>.desktop` あるいは `/usr/share/applications/<app>.desktop` を作成し、次のような内容を書き込む。

```
[Desktop Entry]
Name=Application
Commenct=Comment
Exec=/path/to/executable
Terminal=false
Type=Application
```

`Icon` や `Categories` も指定できるようである。

## 参考

毎回お世話になる ArchWiki。

[デスクトップエントリ-ArchWiki](https://wiki.archlinux.jp/index.php/%E3%83%87%E3%82%B9%E3%82%AF%E3%83%88%E3%83%83%E3%83%97%E3%82%A8%E3%83%B3%E3%83%88%E3%83%AA)

