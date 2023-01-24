Kubernetes 1.24 で Dockershim が廃止となったので、コンテナランタイムを containerd に変更した。

## 手順

手順としては公式ドキュメントに従えば問題ないが、おそらく自分の環境依存で、いくつか気になるところがあった。

[Changing the Container Runtime on a Node from Docker Engine to containerd](https://kubernetes.io/docs/tasks/administer-cluster/migrating-from-dockershim/change-runtime-containerd/)

### `kubectl drain <node>` が正しく完了しなかった

kubernetes-dashboard の Pod を削除できないことが原因で `kubectl drain` に失敗しており、手動で kubernetes-dashboard の Deployment を削除したところ問題なく作業を進めることができるようになった。

### containerd の cgruop ドライバー

手元の環境では `containerd config default` で出力されるものをそのまま使用すれば問題なかった。`[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` に `SystemdCgroup = true` を追記せずとも動作した。

### `kubeadm upgrade apply v1.24.1` で警告が出た

`kubeadm upgrade apply v1.24.1` を実行したところ、`initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future` のエラーが出力された。`/var/lib/kubelet/kubeadm-flags.env` には `--container-runtime-endpoint=unix:///run/containerd/containerd.sock"` のように URL スキームを記述しており、心当たりがないので気にせずアップグレードした。次回のパッチアップグレードでこの警告が消えるかどうか確認したい。

## kube-apiserver に接続できない

この問題については原因が不明である。6443 番ポートが Connection refused となり、クラスタが全く機能しなくなってしまう問題が発生していた。iptables のルールを見たところ、すべてのパケットを DROP する設定になってしまっていた (おそらく Kubernetes の初期状態) ことが理由であると考えている。kube-proxy の ConfigMap を書き換え、 mode を ipvs に設定してみたところ、動作するようになった。正しい対処法かどうかは分からず。
