Office Deployment Tool (Office 展開ツール) を使用することで、特定の Office アプリのみインストールできる。

## ドキュメント

[https://aka.ms/ODT](https://docs.microsoft.com/ja-jp/deployoffice/overview-office-deployment-tool)

## 設定ファイルのサンプル

Excel, Word, PowerPoint のみをインストールするときの例。

```
<Configuration>
    <Add>
        <Product ID="O365ProPlusRetail">
            <Language ID="ja-jp"/>
            <ExcludeApp ID="Access"/>
            <ExcludeApp ID="Groove"/>    <!-- Onedrive for Business -->
            <ExcludeApp ID="Lync"/>      <!-- Skype for Business -->
            <ExcludeApp ID="OneDrive"/>
            <ExcludeApp ID="OneNote"/>
            <ExcludeApp ID="Outlook"/>
            <ExcludeApp ID="Publisher"/>
            <ExcludeApp ID="Teams"/>
        </Product>
    </Add>
</Configuration>
```
