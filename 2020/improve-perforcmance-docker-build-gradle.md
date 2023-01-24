## 問題

Docker で `gradle build` を実行すると、`Starting a Gradle Daemon` で 1 分近くかかってしまう。


## 解決法

適当に `gradle clean` などをあらかじめ実行しておく

```diff
  # Dockerfile

  FROM gradle

  COPY build.gradle.kt build.gradle.kt

+ RUN gradle clean

  COPY src src
  RUN gradle build

```

Daemon 起動中の出力を見る限り、依存関係の計算やダウンロード、`build.gradle.kt` のコンパイルに時間がかかっているように見える。一度 `gradle clean` を実行することによって、`gradle build` の実行時にはそれらを回避できる。また、`src/` 以下を変更しただけの場合 `RUN gradle clean` はキャッシュが利用できるため、初回ビルドより後では高速にビルドできる。


## 測定

簡単な Hello, world! コードで検証した。次に記載するファイル以外は IntelliJ IDEA で生成されたものをそのまま使用した。

```
# src/main/kotlin/xyz/comame/test/Main.kt

package xyz.comame.test

fun main() {
    println("Hello,world!")
}
```

```
# Dockerfile

FROM gradle

RUN useradd -m user
USER user
WORKDIR /home/user

COPY gradle.properties gradle.properties
COPY settings.gradle.kts settings.gradle.kts
COPY build.gradle.kts build.gradle.kts

RUN gradle clean

COPY src src
RUN gradle build
```

### 初回ビルド

`RUN gradle clean` を追加した場合、`RUN gradle clean` は 49 秒、`RUN gradle build` は 18 秒かかった。一方、追加しなかった場合、`RUN gradle build` は 56 秒かかった。

### 2 回目のビルド
`src/main/kotlin/xyz/comame/test/Main.kt` の Hello, world! の文字列を変更して再度ビルドした。`docker build` のキャッシュは有効である。

`RUN gradle clean` を追加した場合、`RUN gradle clean` はキャッシュが使用され、`RUN gradle build` は 18 秒かかった。一方、追加しなかった場合、`RUN gradle build` は 1 分 3 秒かかった。

複数回測定を繰り返した場合でも、上記とおおむね同様の結果を得られた。


## 考慮事項

Dockerfile に `USER root` 、`WORKDIR /root` を指定した場合、`RUN gradle build` は高速化されなかった。`root` ユーザーでは効果がないのか、`WORKDIR /root` の場合に問題があるのか、あるいは他に原因があるのか、今回は未検証である。
