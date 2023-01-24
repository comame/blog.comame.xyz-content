以前からやりたいと思っていた自宅シングルノード Kubernetes クラスターをついに構築し、とりあえず動くまで至った。

## やったこととハマったところ

### Kubeadm を使ったクラスタのインストール

基本的には[公式ドキュメントの指示](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)に従っただけ。

#### cgroup ドライバが Docker と Kubelet で異なっていた

`kubeadm init` をしても Kubelet が動いていないというエラーが発生した。Docker は cgroupfs、Kubelet は systemd を使用していたことが原因だった。`/etc/docker/daemon.json` に `"exec-opts": ["native.cgroupdriver=systemd"]` を追記することで解決。

### Pod ネットワークアドオンがうまくインストールできなかった

まだ挙動をよく理解できているわけではないが、現時点では Kubeadm を使用すると flannel が使用される模様。`kubeadm init --pod-network-cidr 10.244.0.0/16` のようにアドレスを指定することで無事に構築できた。

また、Flannel のマニフェストを自分で読み込まないといけないことに気づかず、 CoreDNS の Pod が Pending のままになってしまった。

### MetalLB のセットアップ

`type=LoadBalancer` を使いたかったので、MetalLB のセットアップをした。[IP アドレスの範囲を指定する](https://metallb.universe.tf/configuration/) 場面で少しつまり、ルータの設定が必要なのかと右往左往していた。ネットワークに対する理解の浅さが露呈した (友人に聞いたら MetalLB は ARP を喋るということを教えてもらった)。最終的に、自宅内ネットワークのサブネット内の IP アドレスを指定してみたところ、なぜかうまく動いた。

### Ingress のセットアップ

ingress-nginx を使用した。`svc/ingress-nginx-controller` に `.spec.loadBalancerIP` を指定して、IP を固定化することにした。将来的に困ることがあればまた考えることにする。

### cert-manager のセットアップ

CloudFlare API を使用して、dns-01 で検証するようにした。ClusterIssuer を使って Ingress を別の Namespace に配置しようと思ったが、dns-01 用の Cloudflare API トークンを保存している Secret を見に行けずにチャレンジが失敗するので、結局 Ingress 用の Namespace を用意することにした。

### Docker Registry を構築した

HTTPS でないとエラーが出るため、`comame.dev` をローカルサブネットに向けて、Let's Encrypt で証明書を発行した。Ingress に `nginx.ingress.kubernetes.io/proxy-body-size` のアノテーションを指定しておかないと、大きいイメージをアップロードするときに怒られる。

インターネットからアクセスすることは想定していないものの、Ingress はインターネットから見えてしまうため、Host ヘッダフォージェリを恐れて一応認証を設定しておいた。

### Kubernetes Dashboard を設置してみた

今動かしているクラスターのバージョンは 1.23 らしいので、[ドキュメントに書いてあるコマンド](https://kubernetes.io/ja/docs/tasks/access-application-cluster/web-ui-dashboard/) を愚直に叩くと、絶妙に API リクエストが失敗するものがデプロイされてしまう。[GitHub のリリースページ](https://github.com/kubernetes/dashboard/releases) にバージョンごとの対応表があるが、1.23 の情報はなかったのでとりあえず最新の 2.4.0 をいれておくことにした。

いまのところ、Pod のログが見れて便利だな程度なので、将来的にどうするかは検討したい。

## 考えている運用方針

HTTP サーバは Ingress を、それ以外は `type=LoadBalancer` の Service に `loadBalancerIP` を直接指定して IP を固定化してしまおうと考えている。Node を増やすつもりはないので、当面の間は困らないだろうと妄想している。

## やりたいこと

というよりやらないといけないこと

- cert-manager の導入 (やった)
- 証明書の自動更新

## マニフェストのレポジトリ

<https://github.com/comame/kubernetes-manifests>
