Ubuntu 20.04 で Softether を systemd のサービスとして登録するには、`/etc/systemd/system/vpnserver.service` に次のように記述する。

[Softether のドキュメント](https://ja.softether.org/4-docs/1-manual/7/7.3#7.3.8_.E3.82.B9.E3.82.BF.E3.83.BC.E3.83.88.E3.82.A2.E3.83.83.E3.83.97.E3.82.B9.E3.82.AF.E3.83.AA.E3.83.97.E3.83.88.E3.81.B8.E3.81.AE.E7.99.BB.E9.8C.B2) をほぼ移植しただけだが、今回はロックファイルの作成はしていない。

```
[Unit]
Description=Softether VPN
After=network.target

[Service]
Type=forking
User=root
ExecStart=/usr/local/vpnserver/vpnserver start
ExecStop=/usr/local/vpnserver/vpnserver stop
Restart=always

[Install]
WantedBy=multi-user.target
```
