## 前提条件

- Windows 10 Pro を使用していること (Hyper-V を使用するため)
- ホストが Windows 11 の動作要件を満たしていること (満たしていなくとも動作する可能性はあるが)

## 手順

### Windows 10 の ISO を用意する

現時点では、公式に Windows 11 の ISO を直接入手することができない。なので、Windows 10 をインストールした後、Windows Update で Windows 11 にアップグレードする。

[Download Windows 10 Insider Preview ISO - microsoft.com](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewiso) からダウンロードするか、あるいは [Media Creation Tool](https://www.microsoft.com/ja-jp/software-download/windows10) で ISO イメージを作成する。

### Hyper-V の仮想マシンを作成する

[Windows 11 の動作要件](https://www.microsoft.com/ja-jp/windows/windows-11-specifications#primaryR2) を満たした仮想マシンを作成するため、設定を少し変更する。以下の項目以外については、自動的に Windows 11 の動作要件を満たすと思われるが、適宜変更が必要かもしれない。

- 「セキュリティ」の「トラステッド プラットフォーム モジュールを有効にする」にチェックを入れる
- メモリを 4 GB 以上割り当てる

Hyper-V のグラフィックは WDDM 1.3 にしか対応していないはずだが、今回はインストールを進めることができた。

### Windows 11 にアップグレードする

Windows Insider Program を Dev チャンネルにすると、Windows 11 のアップデートを受け取ることができる。

## ところで

WSL 2 がきちんと動作するかどうかテストしてみたかったが、Hyper-V 内部で仮想化をすることができないようで、インストールに失敗した、残念。
