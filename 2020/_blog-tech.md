{
    "entry": "blog-tech",
    "title": "ブログシステムを新しくした (技術面)",
    "date": "2020-03-11",
    "tags": [ "Blog" ],
    "type": "md"
}

このブログの技術的な要素を解説する。最終的に静的なファイルとして出力されるまでの流れは、概ね次のようになる。

1. ブラウザ上の JavaScript で記事データを読み、ページを生成する
1. Puppeteer でスクリプト実行後のページを保存する
1. 記事データを読み、サイトマップやフィードを書き出す

理念として、ビルドをする前でも完全なページが閲覧できるようにすることを目指した。

## 記事データ

- [archives/entries.json](https://github.com/comame/blog.comame.xyz/blob/master/archives/entries.json)

年毎にディレクトリを分けることにした。

### `archives/entries.json`

すべての記事のメタデータを保存する。ページを生成するときに参照する。

- `entry`: `.html` 拡張子を付けるとファイル名と対応する。
- `title`: ページに表示されるタイトル。
- `date`: `yyyy-mm-dd` の形式。
- `tags`: 文字列の配列。


## フロントエンドでのページ生成

- [index.html](https://github.com/comame/blog.comame.xyz/blob/master/index.html)
- [assets/js/app.js](https://github.com/comame/blog.comame.xyz/blob/master/assets/js/app.js)

記事データを読み込んで、ページを生成する。後に Puppeteer でスクレイピングする際にページが生成し終わったことを確認できるよう、ページの生成後に特別な `<meta>` タグを埋め込んだ。

今回は自前のルーターライブラリを用いたが、React などを使っても問題ないはず。

### 記事の一覧・タグ

`entries.json` を Fetch API で取得し、該当するページの情報を得た。

### 記事ページ

`entries.json` から記事の HTML のファイルパスを取得し、HTML を埋め込んだ。


## Node.js を使ったビルド

- [build.js](https://github.com/comame/blog.comame.xyz/blob/master/build.js)
- [build/comameito_puppeteer-nginx](https://github.com/comame/blog.comame.xyz/blob/master/build/comameito_puppeteer-nginx)

基本的には `main()` を追っていけばよい。Puppeteer が確実に動くようにするため、[Puppeteer に必要な依存関係](https://github.com/puppeteer/puppeteer/blob/master/docs/troubleshooting.md#chrome-headless-doesnt-launch-on-unix), Nginx, Node.jsをインストールした Docker コンテナを事前に用意した。

### `buildMarkdown()`

[markdown-it](https://www.npmjs.com/package/markdown-it) を使って、Markdown で書かれた記事を HTML に変換する。

[追記] API がよりシンプルで、カスタマイズが簡単な [marked](https://www.npmjs.com/package/marked) を使うように変更した。それにより、外部リンクは新しいタブで開くようにした。

[追記] 静的ビルド前でも完全なページが表示できるよう、Markdowm のビルドをフロントエンドに移動した。

### `copyAssets()`

`assets/` を再帰的に潜っていって、画像や CSS、JavaScript などのファイルをそのままコピーする。

### `crawl()`

[Puppeteer](https://github.com/puppeteer/puppeteer) を用いて、実際に見えている Web ページを JavaScript を用いない静的な HTML ファイルに変換した。`index.html` から内部リンクをすべて拾っていく形にしている。Puppeteer では Browser Context でのスクリプトも簡単に実行できるため、あまり苦労しなかった。

`load` イベントと JavaScipt の実行が完了するタイミングが一致しない (非同期処理) ため、スクリプトでのページ生成が終わった時点で追加される `meta` タグを待機してからページを保存する ([`async page.waitForSelector(selector)`](https://github.com/puppeteer/puppeteer/blob/v2.1.1/docs/api.md#pagewaitforselectorselector-options) が便利だった)。

無限ループを防ぐために、保存済みページの URL の Set を受け取るようにした。これは後のサイトマップ生成でも活用されることになった。

読み込みの高速化のため、ページ生成用の JavaScript を削除し、CSS をインラインに展開している。

### `createSitemap()`

`crawl()` で作成した保存済みページの URL のセットから、サイトマップを書き出した。今回は URL を列挙しただけのテキストファイルにした。

### `createFeed()`

`entries.json` から、Atom のフィードを生成する。

[追記] Markdown のビルドが移動した影響で、Puppeteer を使って記事の内容を拾うようにした。


## GitHub Actions

- [.github/workflows/build.yml](https://github.com/comame/blog.comame.xyz/blob/master/.github/workflows/build.yml)

Node.js のビルドは、GitHub Actions で自動的に行うことにした。ビルド用の Docker コンテナを起動し、Node.js でビルドし、生成物を commit するようになっている。


## 公開

GitHub Pages を使うようにした。GitHub Actions でのビルドが終わり、レポジトリに push されると自動的に反映される。
