Chrome のパスワード同期がされなくなる問題を解決した。環境は Windows 版 Chrome 108.0.5359.125 (Stable)。

## 手順を試す前に

手元の環境で解決した手順であり、すべての場合に適用できない可能性があることに気を付ける。

ブラウザのデータをバックアップする。ブックマークのエクスポートは「ブックマーク マネージャ」から、パスワードのバックアップは <chrome://settings/passwords> と <https://passwords.google.com> の両方からエクスポートしておく。

## 発生していた問題

Chrome でパスワードが同期されず、パスワードマネージャが使えない状態になってしまった。<chrome://sync-internals> を見ると、`components/sync/driver/data_type_manager_impl.cc` で `datatype error was encountered: Preexisting controller error on Sync startup` のようなエラーが表示されており、パスワードの同期に失敗していた。

## 手順

同期時のフォーマットエラーなので、エラーになるパスワードを Google のサーバとローカルから削除すれば直るのでは、という発想。

### インポートエラーになるパスワードを削除する

エクスポートしたパスワードの CSV を <chrome://settings/passwords> でインポートし、エラーになる CSV の行を削除する。

### 同期データの削除

Google のサーバに保存されている同期データを削除する。<https://chrome.google.com/sync> から削除する。

Chrome を使用しているすべての端末で、保存されたパスワードを削除する。手順は [ヘルプページ](https://support.google.com/chrome/answer/2392709?hl=ja) に従う。

### Windows を再起動する

Windows を再起動する。

### パスワードをインポートして、同期を再開する

<chrome://settings/passwords> でパスワードをインポートし、Chrome の同期を再開する。
