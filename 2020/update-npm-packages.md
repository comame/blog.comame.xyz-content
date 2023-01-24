次のコマンドを叩く。

```
node -e "console.log(Object.keys(require('./package.json').dependencies).join(' '))" | xargs npm i
```

やっていることは単純で、package.json の `dependencies` のパッケージ名を Node.js で列挙して、 `npm install` の引数に渡している。package.json のバージョン指定をすべて無視して、とりあえず普通に `npm install` したときにインストールされるものにバージョンを揃えることができる。
