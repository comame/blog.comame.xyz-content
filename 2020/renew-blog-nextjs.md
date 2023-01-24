このブログを [Next.js](https://nextjs.org/) で作り直した。以前に動いていたものをほぼ再現できたと思う。

## 嬉しくなった点

### ソースコードが大幅に読みやすくなった

以前は自作ルーター + 巨大 [app.js](https://github.com/comame/blog.comame.xyz/blob/707d9271f0e17eddd231ee7041408566e424bbdc/assets/js/app.js) で全てを行っていたため、これは何ですかと言いたくなるようなコードになっていた。

### アクセス時に読み込まれる CSS が少し小さくなった

以前はページごとに 3 つの CSS ファイルに分割し、minify したものを HTML に埋め込んでいた。今は Next.js がうまいことコンポーネント分割してくれている、らしい。

以前はインラインで記述していたためすべてのアクセスでダウンロード量に影響したが、今は `<link>` タグで埋め込まれているため、初回より後のアクセスでは読み込みが速くなると思われる。ただ、初回アクセスでどちらが速いのかはよく分かっていない。

### GitHub Actions でのビルドが相当速くなった

以前は Puppeteer を使って静的サイトジェネレーションを行っていたが、それを Next.js がやってくれるようになったおかげで、ビルドが相当高速化した。以前は 1 分以上かかっていたものが、今は 30 秒程度で終わるようになった。


### 開発環境でのプレビューが容易になった

`npx next dev` を叩くだけで容易にプレビューできるようになった。


## 私はここで苦しみました

### SSG をすると、`next/link` のリンクが機能しない

これは以前の URL 構造をそのまま保とうとしたことが原因であるため、どちらかといえば私の使い方の問題であろう。Next.js の Dynamic Routing では、各ページの URL に `.html` の拡張子を付与することはできないため、SSG したときにサイト内リンクが尽くリンク切れを起こす問題が発生した。例えば、SSG 後の URL は `/tags/foo.html` であるにも関わらず、サイト内リンクの `href` は `/tags/foo` のままである、といった具合である。

この問題は Next.js の設定を弄ることで容易に解消できる。`next.config.js` で [`trailingSlash: true`](https://nextjs.org/docs/api-reference/next.config.js/trailing-slash) を設定すればよい。しかし、私のブログの URL 構造を保とうとした場合、これでは過去のリンクがすべてリンク切れを起こしてしまう問題があり、この方法は導入できなかった。

そこで、`<Link>` をラップし、SSG を行うときは手動で `<a>` タグを返すようにした。SSG かどうかは環境変数によって判別することとした ([NEXT_PUBLIC_ から始まる環境変数はブラウザからも参照できる](https://nextjs.org/docs/basic-features/environment-variables#exposing-environment-variables-to-the-browser)) 。

参照: https://github.com/comame/blog.comame.xyz/blob/b7b80220685f6c568bcb9cfedf820db953899123/src/lib/link.tsx

### GitHub Pages でうまいことホストされない

以前と同様に GitHub Pages でホストしようと思ったところ、なぜか `docs/_next/` 配下のファイルが 404 を返してしまう問題に遭遇した。これは GitHub Pages に組み込まれている Jekyll が原因であり、`_` から始まるディレクトリ名を持つディレクトリは配信されないようである。

これを解消するために、GitHub Pages で配信するディレクトリの最上位ディレクトリ (このブログでは `<projectroot>/docs/`) に `.nojekyll ファイルを置いた。

どうやらこの方法は GitHub のドキュメントには書かれておらず、[GitHub Blog](https://github.blog/2009-12-29-bypassing-jekyll-on-github-pages/) にしか公式の記事を見つけることができなかった。
