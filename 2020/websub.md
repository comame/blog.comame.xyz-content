この記事は、[pubsubhubbub.appspot.com](https://pubsubhubbub.appspot.com) で Subscribe した時の挙動について書く。

pubsubhubbub.appspot.com は [PubSubHubbub Core 0.4](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html) と Permanent subscriptions を除く [PubSubHubbub Core 0.3](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.3.html) に従っている。

PubSubHubbub とは、Subscriber 側から見た場合は Webhook である。現在は W3C によって [WebSub](https://w3c.github.io/websub/) として公開されている。WebSub は PubSubHubbub Core 0.4 と概ね同じだと思われる (あまり読み込んでいない)。

## PubSubHubbub Core 0.4

仕様を正確に翻訳したものではない。実装する際には元の仕様を参照することをおすすめする。

### 用語

<dl>
    <dt>Topic</dt>
    <dd>HTTP の URL。変更を購読するリソースを一意に表す。</dd>
    <dt>Hub</dt>
    <dd>PubSubHubbub の Subscribing と Publishing をどちらも実装するサーバ。</dd>
    <dt>Subscriber</dt>
    <dd>Topic の更新を受け取るエンティティ。</dd>
    <dt>Subscription</dt>
    <dd>Topic とその更新通知を受け取る Subscriber の一意な関係。Topic URL と Subscriber Callback URL をキーとする。</dd>
    <dt>Subscriber Callback URL</dt>
    <dd>Subscriber が通知を受け取る URL。</dd>
</dl>

参照: [2. Definitions](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#rfc.section.2)

### Subscribing

購読を登録するとき、次のような流れとなる。
1. Subscriber が Hub に Subscription Request を送信する
1. Hub が Subscription Request を検証する (OPTIONAL)
1. Hub が Subscriber に Verificatin Request を送信する
1. Subscriber が Hub に Verification Response を返す

参照: [5. Subscribing and Unsubscribing](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#rfc.section.5)

### Subscription Request

以下のパラメータを `Content-Type: application/x-www-form-urlencoded` で POST する。

<dl>
    <dt>hub.callback</dt>
    <dd>REQUIRED. Subscriber Callback URL。配信通知はこの URL に対して POST される。クエリパラメータは保持される (<a href='https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#rfc.section.5.1.1' rel='noopener' target='_blank'>5.1.1. Subscription Parameter Details</a>)。</dd>
    <dt>hub.mode</dt>
    <dd>REQUIRED. <code>subscribe</code> あるいは <code>unsubscribe</code> のいずれかの値をとる。</dd>
    <dt>hub.topic</dt>
    <dd>REQUIRED. 購読する Topic の URL。</dd>
    <dt>hub.lease_seconds</dt>
    <dd>OPTIONAL. 購読の有効な時間 (秒)。Hub はこの値に従ってもよいし、従わなくともよい。<code>hub.mode</code> が <code>unsubscribe</code> の場合は無視される。</dd>
    <dt>hub.secret</dt>
    <dd>OPTIONAL. 後に Subscriber が更新通知を検証できるようにするための値。詳しくは後述。</dd>
</dl>

既に有効な Subscription がある場合、Hub は Subscription を更新する。Subscription の確認に失敗した場合は、以前の購読状態を継続する。

参照: [5.1. Subscriber Sends Subscription Request](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#rfc.section.5.1)

### Subscription Response

Hub は HTTP `202 Accepted` によって、Subscription Request が検証され、確認されることを示す。エラーが発生した場合、Hub は 4xx または 5xx を返す。

参照: [5.1.2. Subscription Response Details](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#rfc.section.5.1.2)

### Verification Request

Subscription Request が正しいものかを確認するため、Hub は Subscriber Callback URL に以下のクエリパラメータとともに GET リクエストを送信する。

<dl>
    <dt>hub.mode</dt>
    <dd>REQUIRED. Subscription Request に指定した値と同じ値を示す。</dd>
    <dt>hub.topic</dt>
    <dd>REQUIRED. Subscription Request に指定した値と同じ値を示す。</dd>
    <dt>hub.challenge</dt>
    <dd>REQUIRED. Hub によって生成されたランダムな文字列。Subscriber はこの値をレスポンスとして返すことにより、Subscription を確認する。</dd>
    <dt>hub.lease_seconds</dt>
    <dd>REQUIRED/OPTIONAL. <code>hub.mode</code> が <code>subscribe</code> の場合、REQUIRED となる。Hub によって決定された購読の有効な時間 (秒) を表す。</dd>
</dl>

参照: [5.3. Hub Verifies Intent of the Subscriber](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#rfc.section.5.3)

リクエストされた内容が正しい場合、Subscriber は `hub.challenge` の値をレスポンスボディに含め、2xx を返す。リクエストされた内容が正しくない場合、Subscriber は 404 を返す。

その他のステータスコードを返した場合、またはレスポンスボディが `hub.challenge` の値と異なる場合、確認は失敗する。

参照: [5.3.1. Verification Details](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#rfc.section.5.3.1)

### 更新の通知

コンテンツが更新された時、Hub は Subscriber Callback URL に POST リクエストを送信する。

リクエストを受け取ったとき、Subscriber は 2xx を返さなければならない。それ以外のステータスコードを返した場合、通知は失敗したとみなされる (リダイレクトも機能しない)。2xx のレスポンスはあくまで正常に通知を受け取ったことを表すのみであり、通知の内容が正しいことを示すものではない。

リクエストには、RFC5988 に規定される [Link Header](https://tools.ietf.org/html/rfc5988) が付与される。`rel=hub` には Hub の URL を、`rel=self` には Topic の URL が指定される。

参照: [7. Content Distribution](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#contentdistribution)

### 検証可能な更新の通知

Subscription Request に `hub.secret` を含めた場合、Hub はペイロードの HMAC 署名をヘッダに付与する。署名は 40 バイトの 16 進数表示の SHA-1 である。ヘッダの形式は次のようになる。

```
X-Hub-Signature: sha1=<signature>
```

これにより、通知の内容が第三者に偽造されたものではないことを確認できる。

[8. Authenticated Content Distribution](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.4.html#authednotify)

## PubSubHubbub Core 0.3 による追加事項

### メッセージの形式

更新の通知は [Atom](https://tools.ietf.org/html/rfc4287) あるいは [RSS](https://www.rssboard.org/rss-specification) の形式である。

参照: [4.  Atom Details](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.3.html#anchor4)

### Subscription Request

以下のパラメータが追加される。

<dl>
    <dt>hub.verify</dt>
    <dd>REQUIRED. Subscriber が Subscription を確認する方法を指定する。<code>sync</code> あるいは <code>async</code> のいずれの値をとる。</dd>
    <dt>hub.verify_token</dt>
    <dd>OPTIONAL. 指定した場合、Hub はこの値を Verification Request のクエリパラメータに付与する。</dd>
</dl>

参考: [6.1.  Subscriber Sends Subscription Request](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.3.html#anchor5)

### Verification Request

以下のクエリパラメータが追加される。

<dl>
    <dt>hub.verify_token</dt>
    <dd>OPTIONAL. Subscription Request に指定した値をそのまま含む。</dd>
</dl>

以下のパラメータが変更される。

<dl>
    <dt>hub.lease_seconds</dt>
    <dd>OPTIONAL/REQUIRED. 値が含まれていた場合の挙動は PubSubHubbub Core 0.4 と同一である。値が含まれない場合、後述するように Subscription の更新の挙動が変わる。</dd>
</dl>

参照: [6.2.  Hub Verifies Intent of the Subscriber](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.3.html#verifysub)

### 自動的な Subscription の更新

注意: pubsubhubbub.appspot.com では非対応である。

`hub.lease_seconds` が経過する前に、Hub は自動的に Subscription の確認を行う。

参照: [6.3 Automatic Subscription Refresh](https://pubsubhubbub.github.io/PubSubHubbub/pubsubhubbub-core-0.3.html#autorefresh)

## pubsubhubbub.appspot.com の挙動

筆者が自分の環境で確認したものである。仕様に定められた挙動ではないため、将来的に変更される可能性もある。

### Subscription Request の宛先
送信先は `https://pubsubhubbub.appspot.com/subscribe` である。

### Subscription Response のステータスコード

`hub.verify` が `sync` の場合、正常に Subscription Request が受理されたときは 204 No Content を返す。

### Permanent Subscription の非対応

PubSubHubbub Core 0.3 の 6.3 Automatic Subscription Refresh には対応しない。Subscriber は hub.lease_seconds が経過する前に再び Subscription Request を行う必要がある。

## 補足・考察

pubsubhubbub.appspot.com は PubSubHubbub Core 0.4 に加えて PubSubHubbub Core 0.3 も参照するが、Permanent Subscription には非対応であるため、実用上は `hub.verify_token` が追加されることにだけ注意すれば良いと思われる。大抵の場合は `hub.verify` を `sync` として問題ないであろう。

PubSubHubbub Core 0.3 では更新通知のメッセージ形式が Atom または RSS と規定されているが、PubSubHubbub Core 0.4 および WebSub では撤廃されている。これにより、より汎用性の高いプロトコルを目指したと思われる。

また、新しいバージョンでは `hub.verify_token` も撤廃されている。このプロパティによって Verification Request を検証できるというメリットがあったが、これは Subscriber Callback URL に独自のクエリパラメータを付与することで代替できる。あるいは、第三者によって意図しない Subscription を登録されたとしても、更新通知を処理するかどうかは変わらず Subscriber の判断によるため、仕様から削除されたのかもしれない。

Automatic Subscription Refresh の撤廃は、より簡単な Webhook の登録を目指すという観点からは悪手であろう。一方、PubSubHubbub はそもそも RSS や Atom フィードのリアルタイム化を目指したものであること ([Wikipedia](https://ja.wikipedia.org/wiki/WebSub) を参照のこと) を踏まえれば妥当であるともいえる。もし Subscriber が Subscription の更新を忘れてしまったとしても、最新の更新情報は元のリソースを参照することで簡単に取得できるからである。

## 経緯 (余談)

YouTube Data API v3 について調べていたのだが、Search: list (100 Units/Request) をむやみに叩くとすぐに制限 (10,000 Units/Day) に達してしまう。特定のチャンネルのアップデートを素早く受け取るために、[PubSubHubbub をサポート](https://developers.google.com/youtube/v3/guides/push_notifications)していた。
