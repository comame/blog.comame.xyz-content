JavaScript で TOTP を実装してみたので、その実装メモなど。

https://github.com/comame/TOTP

## TOTP

RFC 6238: TOTP: Time-Based One-Time Password Algorithm で規定される。Google Authenticator アプリなどを使って時刻ベースで生成される (大抵) 6 桁のワンタイムパスワードであり、2 要素認証に用いられる。

疑似コードで次のように表される。

```
K: 共有シークレット、QR コードとかで読み込むやつ
X: ステップ秒。大抵 30 秒。この値の周期でトークンが切り替わる
T0: Unix Time の開始秒。大抵 0

T = (Current Unix Time - T0) / X) as Integer

TOTP(K, T) = HOTP(K, T) = Truncate(HMAC-SHA-1(K, T))
```

トークンの生成アルゴリズムには HOTP を使用する。

のちに記述するように、HOTP では T の値が 32-bit までしかサポートされていないため、4.2 節では 32-bit より大きい整数をサポートするように規定されている。具体的にどう実装するのかは記述されていないので、Google Authenticator の実装などを見るのが良いと思われる。

## HOTP

RFC 4226: HOTP: An HMAC-Based One-Time Password Algorithm で規定される。カウンターベースのワンタイムパスワード。

```
K: 共有シークレット
C: カウンター。8-byte の整数
HS: 20-byte
S: 31-bit
D: 桁数が Digit の HOTP トークン

DT(HS: bytes[20])
    OffsetBits = HS[19] & 0xF // HS[19] の下位 4-bit
    Offset = OffsetBits as Integer // 0 <= offset<= 15
    P = HS[Offset]...HS[Offset + 3]
    return P & 0x7FFFFFFF // 下位 31-bit

HS = HMAC-SHA-1(K, C)
S = DT(HS)
D = (S as Integer) mod 10^Digit
```

RFC の 5.4 節の例は次の通りである。

```
HS = {
    1F, 86, 98, 69, 0E,
    02, CA, 16, 61, 85,
    50, EF, 7F, 19, DA,
    8E, 94, 5B, 55, 5A
}

OffsetBits = 0xA
Offset = 10
P = 0x50EF7F19

S = 0x50EF7F19 & 0x7FFFFFFF = 0x50EF7F19
D = 872921
```

## HMAC

https://www.ipa.go.jp/security/rfc/RFC2104JA.html に日本語での解説がある。

```
H: ハッシュ関数
K: シークレット
M: メッセージ

ipad = 0x3636...
opad = 0x5C5C...
// ipad, opad の長さはハッシュ関数のブロック長 (SHA-1 の場合 512-bit) と同一

||: ビットの連結

HMAC(K, M) = H((K xor opad) || H((K xor ipad) || M))
```

K と opad, ipad とで排他的論理和をとっていることから分かるように、K の長さもハッシュ関数のブロック長と同一にする必要がある。K は入力値であることから、長さを揃えるために次のような処理を順に行う。

K の長さがブロック長より大きい場合、K をハッシュ関数に通す。K の長さがブロック長より小さい場合、ブロック長と同一になるまで末尾に 0 を追加する。


## SHA-1

https://www.ipa.go.jp/security/rfc/RFC3174JA.html に日本語での解説がある。

まずはメッセージのパディングを行い、メッセージ長を 512-bit の倍数にする。オリジナルのメッセージ長が 512-bit の倍数であった時にもパディングは行う。パディングは次の手順で行う。

- 末尾に 1-bit の 1 を付加する
- メッセージ長が mod 512-bit = 448 になるまで 0 を末尾に付加する
- オリジナルのメッセージ長を 64-bit で付加する


次の関数を定義する。

```
f(t; B, C, D) = (B & C) | ((~B) & D); (0 <= t <= 19)
f(t; B, C, D) = B ^ C ^ D; (20 <= t <= 39)
f(t; B, C, D) = (B & C) | (B & D) | (C & D) (40 <= t <= 59)
f(t; B, C, D) = B ^ C ^ D; (60 <= t <= 79)

K(t) = 0x5A827999; (0 <= t <= 19)
K(t) = 0x6ED9EBA1; (20 <= t <= 39)
K(t) = 0x8F1BBCDC; (40 <= t <= 59)
K(t) = 0xCA62C1D6; (60 <= t <= 79)

S^N(n) = (M << n) | (M >> (32 - n)) // 循環左シフト
A + B = (A + B) mod (2 ** 32)
```

複数の計算方法があるようだが、今回は次の方法で計算した。

```
M: 512-bit ごとに区切ったメッセージ

H0 = 0x67452301
H1 = 0xEFCDAB89
H2 = 0x98BADCFE
H3 = 0x10325476
H4 = 0xC3D2E1F0

for each M(i)
    W[16] = M(i) // M(i) を 32-bit ごとに分割する

    for t from 16 to 79
        W[t] = S^1(W[t - 3] ^ W[t - 8] ^ W[t - 14] ^ W[t - 16])

    A = H0; B = H1; C = H2; D = H3; E = H4

    for t from 0 to 79
        TMP = S^5(A) + f(t; B, C, D) + E + W[t] + K(t)
        E = D; D = C; C = s^30(B); A = TMP

    H0 += A; H1 += B; H2 += C; H3 += D; H4 += E

// H0 H1 H2 H3 H4 をこの順で並べる
```


## 実装で苦しんだところ (大体うっかり)

### HOTP

C は 8-byte 整数。

### HMAC

ハッシュ関数に SHA-1 を使用する場合、ブロック長は 64-byte、出力長は 20-byte であることから、K にハッシュを通した後に 0 を追加する処理を行う必要がある。

### SHA-1

(追記あり) 左循環シフト。`(n: number, x: number) => (x << n) | (x >> (32 - n))` と素直に書くと、x は 32-bit 整数、`0 <= n < 32` であることからオーバーフローする。


## 参考にしたもの

<dl>
    <dt>IPA の RFC 日本語訳</dt>
    <dd>HMAC と SHA-1 の実装</dd>
    <dt>Golang のソースコード</dt>
    <dd>テストケースや各アルゴリズムの実装。読みやすい。</dd>
    <dt>RFC</dt>
    <dd>テスト用のサンプルケースが Appendix に書いてあることを初めて知った</dd>
</dl>


## 左循環シフトの高速化

https://github.com/comame/TOTP/compare/e495732d9156e805c78ff2626e95f09301123b32...361549fc64a207bcc402f48f802ee4d68b61cac6

オーバーフローをしないためにビットの配列に変換してシフト操作をしていたものを、number のまま計算できるように書き換えた。オーバーフローを起こさないため、32-bit 整数を 8-bit ずつに区切り、場合分けした。また、シフト演算子を使うとオーバーフローを起こすため、掛け算と割り算でシフト演算子と同等の計算を行うようにした。

次のようなコードで 10 回ずつ計測しそれぞれ平均をとったところ、高速化前は 616 ms、高速化後は 5 ms となった。

また、SHA-1 の計算自体も、おおむね半分から 5 分の 1 程度の実行時間に短縮された。

```
const t = Date.now()
for (let i = 0; i < 100000; i +=1) circularShift(16, 0x12345678)
console.log(Date.now() - t)
```

```
測定結果

高速化前 [ms]
    608 605 611 598 612 599 698 605 612 611
高速化後 [ms]
    4   4   6   4   5   5   5   4   5   5
```
