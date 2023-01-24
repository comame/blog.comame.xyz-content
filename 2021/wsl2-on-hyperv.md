Hyper-V にインストールしている Windows 11 で WSL 2 を動かすことができた。

## 手順

### 1. Hyper-V の管理機能を有効にする

「Windows の機能の有効化または無効化」で Hyper-V > Hyper-V 管理ツール > Windows PowerShell 用 Hyper-V モジュール を有効にする。

参考: <https://docs.microsoft.com/ja-jp/skypeforbusiness/troubleshoot/server-install-or-uninstall/get-vm-not-recognized-during-install-cloud-connector>

### 2. Hyper-V での入れ子になった仮想化を有効にする

PowerShell で以下のコマンドを実行する。`<VMName>` は `Get-VM` で確認できる。

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```

WSL 2 は Hyper-V を使用しているため、この方法で仮想マシン内で仮想環境を使用できる。

参考: <https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/user-guide/nested-virtualization>
