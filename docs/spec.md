# QRコードジェネレーター 構築仕様書

## 概要

URLまたは任意テキストからQRコードを生成し、PNG画像としてダウンロードできるWebアプリを構築する。

---

## 技術スタック

| 役割 | 採用技術 |
|------|----------|
| バックエンド | Python / FastAPI |
| テンプレート | Jinja2 |
| フロントエンド | HTML + TailwindCSS (CDN) + Vanilla JS |
| QR生成ライブラリ | `qrcode[pil]` (Python) |
| 画像処理 | Pillow |
| サーバー | Uvicorn |

---

## ディレクトリ構成

```
qr-generator/
├── main.py
├── requirements.txt
├── templates/
│   └── index.html
└── static/
    └── (生成QR画像の一時保存 or 不要)
```

---

## タスク一覧

### Task 1: プロジェクト初期化

- `requirements.txt` を作成する
  ```
  fastapi
  uvicorn
  qrcode[pil]
  pillow
  python-multipart
  jinja2
  ```
- 仮想環境を作成し依存をインストールする
  ```bash
  python -m venv .venv
  source .venv/bin/activate
  pip install -r requirements.txt
  ```

---

### Task 2: バックエンド実装 (`main.py`)

以下のエンドポイントを実装する。

#### `GET /`
- `templates/index.html` を返す

#### `POST /generate`
- リクエストボディ（フォームデータ）:
  - `text`: QRコードに埋め込む文字列（必須）
  - `size`: QRコードの1モジュールあたりのピクセル数（任意、デフォルト: `10`）
  - `border`: 余白のモジュール数（任意、デフォルト: `4`）
- 処理:
  1. `qrcode` ライブラリでQRコードを生成
  2. PNG画像をバイト列として生成（ファイル保存不要）
  3. `StreamingResponse` でPNG画像を返す
- レスポンスヘッダー:
  - `Content-Type: image/png`
  - `Content-Disposition: attachment; filename="qrcode.png"` （ダウンロード用）

実装例:
```python
import io
import qrcode
from fastapi import FastAPI, Form
from fastapi.responses import StreamingResponse, HTMLResponse
from fastapi.templating import Jinja2Templates
from fastapi.requests import Request

app = FastAPI()
templates = Jinja2Templates(directory="templates")

@app.get("/", response_class=HTMLResponse)
async def index(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})

@app.post("/generate")
async def generate(
    text: str = Form(...),
    size: int = Form(10),
    border: int = Form(4),
):
    qr = qrcode.QRCode(
        version=None,
        error_correction=qrcode.constants.ERROR_CORRECT_M,
        box_size=size,
        border=border,
    )
    qr.add_data(text)
    qr.make(fit=True)
    img = qr.make_image(fill_color="black", back_color="white")
    buf = io.BytesIO()
    img.save(buf, format="PNG")
    buf.seek(0)
    return StreamingResponse(
        buf,
        media_type="image/png",
        headers={"Content-Disposition": 'attachment; filename="qrcode.png"'},
    )
```

---

### Task 3: フロントエンド実装 (`templates/index.html`)

以下のUIを実装する。

#### UI要素

- テキスト入力欄（URL or テキスト）
- サイズ調整スライダー（box_size: 5〜20）
- 「生成」ボタン
- QRコード画像のプレビュー表示エリア
- 「ダウンロード」ボタン（生成後に表示）

#### 動作仕様

1. 「生成」ボタンをクリック
2. `fetch` で `POST /generate` を呼び出す（`FormData` で送信）
3. レスポンスの `blob()` を取得し、`<img>` タグの `src` に `URL.createObjectURL(blob)` をセット
4. プレビューを表示する
5. 「ダウンロード」ボタンは `<a download="qrcode.png" href="...">` で実装し、生成後に有効化

#### スタイル

- TailwindCSS CDN を使用
- レスポンシブ対応（スマホでも使える）
- シンプルで清潔感のあるデザイン

---

### Task 4: 動作確認

以下のケースでテストする。

| テストケース | 期待結果 |
|---|---|
| 通常のURL (`https://example.com`) を入力して生成 | QRコードが表示され、スマホ等で読み取れる |
| 日本語テキストを入力して生成 | 文字化けなく読み取れる |
| 空文字で生成ボタン押下 | バリデーションエラーを表示 |
| ダウンロードボタン押下 | `qrcode.png` がローカルに保存される |
| スライダーを最大にして生成 | 大きなQRコードが生成される |

---

## 起動方法

```bash
# 依存インストール（初回のみ）
pip install -r requirements.txt

# サーバー起動
uvicorn main:app --reload --port 8000
```

ブラウザで `http://localhost:8000` を開く。

---

## Claude Code へのプロンプト

以下をそのままコピーして Claude Code に貼り付ける。

---

```
以下の仕様書に従って、QRコードジェネレーターWebアプリを実装してください。

## 要件

- FastAPI + Jinja2 のバックエンド
- `qrcode[pil]` ライブラリでサーバーサイドQRコード生成
- フロントエンドは HTML + TailwindCSS (CDN) + Vanilla JS
- ファイル構成:
  - main.py（FastAPIアプリ本体）
  - templates/index.html（フロントエンド）
  - requirements.txt

## エンドポイント

1. GET / → index.html を返す
2. POST /generate → フォームデータ（text, size, border）を受け取り、PNG画像をStreamingResponseで返す

## フロントエンドの動作

1. テキスト入力欄にURLまたはテキストを入力
2. サイズ調整スライダー（box_size: 5〜20）
3. 「生成」ボタンを押すと fetch で POST /generate を呼び出し、レスポンスのblobをimgタグに表示
4. 「ダウンロード」ボタンで qrcode.png として保存
5. 入力が空の場合はアラートを表示

## スタイル

- TailwindCSS CDN を使用
- シンプルで清潔感のあるデザイン
- スマホでも使えるレスポンシブ対応

## 起動確認

実装完了後、以下を実行して動作確認してください:
uvicorn main:app --reload --port 8000
```

---

## 参考ライブラリ

- FastAPI: https://fastapi.tiangolo.com/
- qrcode: https://github.com/lincolnloop/python-qrcode
- TailwindCSS CDN: https://cdn.tailwindcss.com
