PowerShell スクリプト (.ps1) をコマンドプロンプト (あるいはバッチファイル) から管理者権限で実行したかった。

次のようなスクリプトを置いておく。

```
# sudo.ps1

start-process powershell -ArgumentList @("-ExecutionPolicy", "Bypass", "-File", $Args[0]) -verb runas
```

呼び出すときは次のようにする。

```
powershell -ExecutionPolicy Bypass -File path\to\sudo.ps1 path\to\script.ps1
```
