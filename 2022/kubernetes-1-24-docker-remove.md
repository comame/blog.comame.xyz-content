Docker サポートが削除される警告をきちんと読まずに Kubernetes を 1.24 にアップグレードしたらクラスタが壊れたという話。

## 経緯

### `kubeadm upgrade plan` に警告がでる

`apt update` をしたところ Kubernetes 1.24 へのアップグレードができそうだったので、`kubeadm upgrade plan` を実行したところ、次のような警告が出た。

```
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0505 14:28:42.708621  126359 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/var/run/dockershim.sock". Please update your configuration!
[preflight] Running pre-flight checks
```

なにやら Docker 周りの警告のようだが、自動でスキームを付けるといったことが書かれているし問題ないだろうとおもってアップグレードを続行した。

### Pod がスケジュールされなくなる

アップグレードが終わった直後から、HTTP が繋がらないというアラートが大量に飛んでくる。`kubectl describe pod` をしてみると、どうやら Node に Pod をスケジュールできないことが原因の模様。

`kubectl describe nodes` をすると、なぜか `NodeNotReady` の文字が。

### 原因の特定

Kubelet がうまいこと動いていないのだろうと当たりをつけ、Kubelet のログを確認する。そうすると、なにやら次のようなエラーを吐いて起動に失敗していることが判明した。

```
Error: failed to parse kubelet flag: unknown flag: --network-plugin
```

`/var/lib/kubeadm-flags/env` を見ると、(当時のファイルが残っていないので定かではないが) 確かに `--network-plugin=cni` のようなオプションが付けられていた。

ここで、はじめに `kubeadm upgrade plan` したときの警告に含まれていた `dockershim.sock` やオプションに含まれる `CNI` から、そういえば Kubernetes が Docker のサポートを打ち切るというような記事を以前に読んだことを思い出す。

慌ててリリースノートを読むと、確かに 1.24 から Docker をそのまま使い続けることができないことが書かれていた。

### 解決

[FAQ](https://kubernetes.io/blog/2022/02/17/dockershim-faq/#can-i-still-use-docker-engine-as-my-container-runtime) を読むと 、Docker のまま今回のアップデートに対応する方法が書かれていることに気がついた。

思い切って `containerd` に差し替えることも考えたが、とりあえずの対策として `cri-dockerd` をインストールして解決することにした。


### その後

```
Flag --pod-infra-container-image has been deprecated, will be removed in 1.27. Image garbage collector will get sandbox image information from CRI.

 "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
 ```

のような警告も出ていたので、該当のオプションも消した。

## 教訓

- リリースノートを読むこと (特にマイナーアップデートをするときは)
- 警告をおろそかにしないこと

当然のことである。
