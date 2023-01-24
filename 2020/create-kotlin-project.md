Gradle 周りの設定の備忘録。

## Gradle Wrapper

Gradle Wrapper を準備する。

```
$ gradle wrapper
```

## ビルド設定

`gradle jar` で実行可能な Jar ファイルを吐けるようにする。

```diff
// build.gradle.kts

  plugins {
+   application
  }

+ application {
+     mainClassName = "xyz.comame.project.MainKt"
+ }

+ val jar by tasks.getting(Jar::class) {
+     manifest {
+         attributes["Main-Class"] = "xyz.comame.project.MainKt"
+     }
+ }
```

## ソースコード

`src/main/kotlin/xyz/comame/project/` に置く。

## 最後に

正直よく分かっていない。`gradle init` を試してみたら、実はこれでもプロジェクト設定をしてくれるらしい。
