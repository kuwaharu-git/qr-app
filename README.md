# qr-app

QRコード生成・管理Webアプリケーション

## 技術スタック

- **バックエンド**: FastAPI (Python)
- **フロントエンド**: Vite + React (TypeScript)
- **Webサーバ (本番)**: Nginx
- **コンテナ**: Docker / Docker Compose

## 開発環境

### 起動方法

```bash
docker-compose --profile dev up --build
```

### アクセスURL

- フロントエンド (Vite開発サーバ): http://localhost:5173
- バックエンド (FastAPI): http://localhost:8000
- API ドキュメント: http://localhost:8000/docs

### 停止方法

```bash
docker-compose --profile dev down
```

## 本番環境

### ビルド・起動方法

```bash
docker-compose --profile prod up --build
```

### アクセスURL

- アプリケーション全体: http://localhost:8080
  - 静的ファイル: Nginxが配信
  - APIリクエスト (`/api/*`): FastAPIへプロキシ

### 停止方法

```bash
docker-compose --profile prod down
```

## プロジェクト構成

```
.
├── backend/          # FastAPI アプリケーション
│   ├── Dockerfile
│   └── app/
│       └── main.py
├── frontend/         # Vite + React アプリケーション
│   ├── Dockerfile
│   ├── app/          # フロントエンドソースコード
│   └── nginx/        # Nginx 設定 (本番環境用)
└── docker-compose.yml
```