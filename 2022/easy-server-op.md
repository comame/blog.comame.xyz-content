個人向け落ちてもどうでもいいようなサーバの超手抜き運用。

## 落ちたときに気づけるようにする

[Google Apps Scirpt](https://developers.google.com/apps-script) で HTTP リクエストを叩くだけのスクリプトを書き、1 分おきのトリガーを設定する。エラー時のメール通知を即時にしておけば、落ちた時にメールが飛んでくる。

(補足)

`GmailApp.sendEmail()` を使わずとも、失敗時に例外を投げておけば、トリガーの失敗が自動的にメール通知される。 

## やらかしたときに対処できるようにする

[Chrome Remote Desktop](https://remotedesktop.google.com) を入れておく。これは間違えて SSH を落としても、ファイアウォールの設定を間違えても (多分)、端末がインターネットに繋がっていれば繋がる。

出先で VPN に繋がらなくなって唸っていたら、[yasu0796](https://twitter.com/yasu0796) さんにアドバイスを頂いてこの方法に気づいた。
