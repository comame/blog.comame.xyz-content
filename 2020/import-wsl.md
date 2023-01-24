1. `wsl --export ubuntu-20.04 <アーカイブのパス>` でエクスポート
1. Ubuntu-20.04 のストアアプリをアンインストールしておく
1. `wsl --import ubuntu-20.04 <インストール場所> <アーカイブのパス>` でインポート
1. Microsoft Store から Ubuntu-20.04 をインストール
1. `ubuntu2004 config --default-user <user>` でデフォルトユーザーを普段のものに変更

少なくとも現時点では、ほぼ公式ドキュメントに書いてある手順そのまま。

## なぜこうなるか

既に同じ名前のディストリビューションがインストールされている場合 `wsl --import` で弾かれるため、インポート時にはアンインストールしておく必要がある。アンインストールせずとも `wsl --unregister <distro>` だけで良いかどうかは未検証。

`wsl --import` だけでは `ubuntu2004` コマンドが使用できず、デフォルトユーザーを root から変更できない。どうやら Microsoft Store からインストールしたディストリビューションでないとディストリビューション名のコマンドは使用できないようであるため、改めて Microsoft Store からディストリビューションをインストールする。この時、ストアアプリとしてインストールされる Ubuntu のディストリビューション名も `ubuntu-20.04` だが、インポートしたデータは上書きされない。

## 参考

- [配布の既定のユーザーを変更する, Linux ディストリビューションの管理 | Microsoft Docs](https://docs.microsoft.com/ja-jp/windows/wsl/wsl-config#change-the-default-user-for-a-distribution)
