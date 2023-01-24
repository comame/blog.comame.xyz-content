## ロック中に Windows がスリープする

通常、ロック画面で放置しておくと勝手にスリープしてしまうようなのだが、この挙動で困ってしまうので、次の方法で対処した。


1. regedit.exe で `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\7bc4a2f9-d8fc-4469-b07b-33eb785aaca0` の Attribute を 2 にする
1. 電源オプションの詳細設定で「スリープ」-「システム無人スリープタイムアウト」の時間を適当に設定する

## ロック中にモニターが切れる

同様に、スリープ中にモニターが切れる問題。

1. `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\7516b95f-f776-4464-8c53-06167f40cc99\8EC4B3A5-6868-48c2-BE75-4F3044BE88A7` の Attribute を 2 にする
1. 「ディスプレイ」-「コンソールロックディスプレイオフのタイムアウト」を適当に設定する
