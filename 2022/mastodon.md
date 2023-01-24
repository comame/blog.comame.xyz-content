## モチベーション

Twitter のスレッドや Zenn のスクラップのような書き心地で、適当に書き殴れるメモ帳が欲しかった。そういう気持ちがあったところに、Twitter 騒動で Mastodon の話題を耳に挟んだので、じゃあやってみるかの気持ちでインスタンスを立ててみた。

## やったこと

- Mastodon のインスタンスを Kubernetes の上にデプロイした
- OpenID Connect でユーザー認証ができるようにした

### Mastodon の構築

Mastodon の GitHub レポジトリにおいてある、[docker-compose.yml](https://github.com/mastodon/mastodon/blob/main/docker-compose.yml) が参考になった。docker-compose.yml をもとに Deployment を作り、その後はコンテナ内で `RAILS_ENV=production bundle exec rake mastodon:setup` を叩くだけだった ([セットアップのドキュメント](https://docs.joinmastodon.org/admin/install/#generating-a-configuration))。

SMTP サーバの設定は、localhost にしても特に問題は起きなかった。当然メール送信には失敗するが、失敗したメールのログは Sidekiq で閲覧できるので、個人用途ならあまり問題になることはなさそうだと思っている。

Admin ユーザーの作成を行うかも問われるが、このバージョンでは Redis への接続に失敗するので、あとから自分で作成する必要があった。ユーザーを作成するには、コンテナ内で `bundle exec tootctl accounts create ...` のようにすればよい。`role=Owner` を指定する必要があったが、`Owner` の値はまだドキュメント化されていない様子。

### OpenID Connect のセットアップ

このバージョンではまだドキュメントが揃っていなかったため、該当する [GitHub の Issue](https://github.com/mastodon/mastodon/issues/7958#issuecomment-1308525637) を頼りに設定した。メールアドレスの情報がないと認証が通らないため、 `scope` の値に `openid,email,profile` を指定しないと動かないようである。メールアドレスが OP から渡されなかったときでもエラーにせず、Mastodon 側で改めてメールアドレスの設定ができると良いのになあと思いながら設定していた。

## 得られたこと

- 例えばブログの草稿を雑に書きなぐることができるようになった
  - [スレッド](https://mastodon.comame.xyz/@comame/109377430498349331)
- 使い方によっては、その時考えていたことを時系列に従って思い返すことができそう
