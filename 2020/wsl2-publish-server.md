Hyper-V の NAT をこねくり回そうとしたけれども、うまくいかなかった。`netsh` コマンドを使うことで、簡単にポート変換ができる。

## Workaround

[Issue #4150 microsoft/WSL | GitHub](https://github.com/microsoft/WSL/issues/4150#issuecomment-504209723)

```
# PowerShell (Administrator)

> netsh interface portproxy add v4tov4 \
    listenport=<External Port> \
    listenaddress=<External Address> \
    connectport=<Internal Port> \
    connectaddress=<Internal Address>
```

### connectaddress

WSL 2 に割り当てられた内部 IP を指定する必要がある。WSL 2 内で

```
# bash (WSL 2)

$ ip a show eth0
```

を叩くと取得できる。


## netsh interface portproxy

[Netsh interface portproxy コマンド | Microsoft Docs](https://docs.microsoft.com/ja-jp/windows-server/networking/technologies/netsh/netsh-interface-portproxy)

## サンプルスクリプト

```
$port = Read-Host 'Port'
$internal_ip = bash.exe -c "ip a show eth0 | grep -E '^\s*inet ' | xargs echo | cut -d ' ' -f 2 | cut -d '/' -f 1"

netsh interface portproxy delete v4tov4 listenport=$port listenaddress=0.0.0.0
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$port connectaddress=$internal_ip connectport=$port

```
