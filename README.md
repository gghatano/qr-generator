# QRコード生成ツール

ブラウザ上でQRコードを生成・ダウンロードできるWebツールです。

https://gghatano.github.io/qr-generator/

## 機能

- URLやテキストからQRコードを生成
- サイズ調整（スライダー）
- PNG画像としてダウンロード

## 技術スタック

- HTML / JavaScript（サーバー不要）
- [qrcode-generator](https://github.com/nicohman/qrcode-generator) - QRコード生成ライブラリ
- [Tailwind CSS](https://tailwindcss.com/) - スタイリング
- GitHub Pages + GitHub Actions - ホスティング・デプロイ
