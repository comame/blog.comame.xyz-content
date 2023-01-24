開発マシンの 8080 番ポートに、インターネットから https://dev-8080.comame.xyz のような形でアクセスできるようにした。

## やり方

### サーバの sshd の設定をいじる

[GatewayPorts](https://linux.die.net/man/5/sshd_config#:~:text=GatewayPorts) の設定をいじっておく。これによって、サーバの 0.0.0.0 で転送されたポートを待ち受けることができるようになる。

```
GatewayPorts=clientspecified
```

### ロードバランサーを設定する

<https://github.com/comame/kubernetes-manifests/commit/8a9d189ed4a6f3158798531db015389cc0dc548c>

### サーバに対して SSH ポートフォワードをする

開発マシンからサーバに対して SSH 接続する。

```
$ ssh -R 0.0.0.0:8080:localhost:8080 <server>
```

## 感想とか

開発中のものにインターネットからアクセスする必要があったときにデプロイしなくて済むようになったので、開発体験がよくなった。ロードバランサが動いているマシンに対して SSH 接続をしないといけないので、スケールはしなさそう？

アクセス制御はいずれやりたい
