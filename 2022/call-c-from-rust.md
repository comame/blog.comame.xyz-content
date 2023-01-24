Rust のコードから C を呼び出して Hello, world! するだけ。

## 手順
### C のコードを書いてコンパイルする

適当にコードを書く。

```
// /project_root/lib/hello_world.c

#include "stdio.h"

void hello_world() {
    printf("Hello, world!");
}
```

これをコンパイルして、静的ライブラリにする。ライブラリのファイル名の先頭には `lib` を付ける必要がある。

```
$ gcc -c hello_world.c -o hello_world.o
$ ar crs libhello_world.a hello_world.o
```


### Rust から呼び出す

`link` attribute でライブラリを指定する。`libhello_world` を使いたいので、`#[link(name="hello_world")]` とする。build.rs で `cargo:rustc-link-lib=hello_world` を出力してもよい。

```
// /project_root/src/main.rs

#[link(name="hello_world")]
extern {
    fn hello_world();
}

fn main() {
    unsafe {
        hello_world();
    }
}
```

[参考 (the book)](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code), [参考 (the 'nomicon)](https://doc.rust-lang.org/nomicon/ffi.html)

ライブラリのファイルがある場所を指定する。

```
// build.rs

fn main() {
    println!("cargo:rustc-link-search=native=/project_root/lib");
}
```

[参考 (cargo:rustc-link-search)](https://doc.rust-lang.org/cargo/reference/build-scripts.html#rustc-link-search)

### 動かす

```
$ cargo run
```

## ちょっとした疑問点

- the book には `extern "C" { }`、the 'nomicon には `extern { }` と書かれていたが、何が違うのか
    - C を Rust から呼び出すときはどちらでもよくて、Rust を C から呼び出すときに影響してくる？
    - <https://stackoverflow.com/questions/44056461/difference-externc-vs-extern/44056776>

## 感想

C のことはよくわかっていないが、思ったよりも簡単に動かすことができた。実際には [cc](https://crates.io/crates/cc) を使うべきだと思われる。

大抵のことは既存の Crate が用意されていると思うので、自分で使うことは当分の間なさそう。

## 参考

- <https://blog.ojisan.io/rust-ffi-cpp-wakaran/>
- <https://www.yunabe.jp/docs/static_library.html>
