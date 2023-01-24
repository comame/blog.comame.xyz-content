Kubernete Dashboard に OpenID Connect でログインしたかったというメモ。

まずは、[Kubernetes Dashboard のドキュメント](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md) に従って、Service Account を作っておく。

要するに、Kubernetes の認証を通すため、特定の Service Account に対する ID Token を発行できれば良い。

<https://kubernetes.io/ja/docs/reference/access-authn-authz/authentication/#openid-connect%E3%83%88%E3%83%BC%E3%82%AF%E3%83%B3> に従って、kube-apiserver を Relying Party として設定する。

ID Token に含まれるユーザー ID のクレームに、そのまま Service Account の名前を指定すると、認証できない。そこで、ユーザー名を `system:serviceaccount:<namespace>:<name>` のようにしてやると、サービスアカウントとして認証してくれる。

`kubectl create token <name>` で取得できる JWT を <https://jwt.io> などに放り込むと、そうなっていることを読み取れる。

## ところで

ID Token を発行できただけなので、そのトークンをどうやって Kubernetes Dashboard に渡すかというのは別問題。例えばプロキシサーバみたいなものを立てる必要があるかもしれない。

どうせ自分ひとりしか使わないなら、`type: kubernetes.io/service-account-token` の Secret を作っておいて、そのトークンをブラウザに覚えさせておけばよいのでは？
