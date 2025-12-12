📘 QRセーフブラウザ Webアプリ – 仕様書

⸻

1. システム概要

本システムは、QRコードから取得した URL に対して即アクセスせず、一度バックエンド側で安全性をチェックしたうえで、ユーザに表示する Webアプリである。
また、読み取ったQRの履歴をユーザごとに保存し、いつでも確認できるようにする。

⸻

2. 技術スタック

◆ フロントエンド（Web）
	•	React（TypeScript）
	•	Vite
	•	react-router-dom
	•	Tailwind CSS
	•	QR読み取りライブラリ：zxing-js / react-qr-reader
	•	react-query（HTTPキャッシュ）

◆ バックエンド（API）
	•	Python 3.11+
	•	FastAPI
	•	Uvicorn
	•	SQLAlchemy
	•	MySQL 8.0
	•	JWT 認証
	•	OpenSSL（証明書情報取得用）

◆ インフラ / その他
	•	Docker / Docker Compose
	•	Nginx（本番プロキシ）
	•	SMTP サーバ（Magic Link 認証メール）

⸻

3. 機能一覧

3.1 認証機能
	•	メールアドレスでログイン（Magic Link）
	•	ログイン時に JWT 発行
	•	ログアウト（トークン破棄）

3.2 QRコード読み取り
	•	カメラ起動
	•	QR / URL文字列を取得
	•	バックエンドに送信

3.3 URL安全性チェック

バックエンドで以下を実行：
	1.	URLの正規化
	2.	HTTPリクエスト（HEAD/GET 最小限）
	3.	短縮URLの解決
	4.	SSL証明書情報取得
	5.	危険URL判定（ルールベース）

3.4 履歴管理
	•	各ユーザのQR結果を保存
	•	一覧、検索、削除

3.5 危険URLレポート
	•	ユーザからの「危険報告」
	•	ブラックリストに追加

⸻

4. API仕様

4.1 認証関連

POST /auth/request-link

Magic Link送信

{
  "email": "user@example.com"
}

レスポンス：

{ "ok": true }


⸻

GET /auth/verify

Magic Link のトークン検証
→ 成功すると JWT 発行

パラメータ：

/auth/verify?token=xxxx

レスポンス：

{
  "token": "JWT_TOKEN",
  "user": { "id": 1, "email": "user@example.com" }
}


⸻

4.2 QR関連

POST /qr/scan

QR文字列を受け取り、安全性チェックを実行

{
  "rawText": "https://example.com"
}

レスポンス例：

{
  "final_url": "https://example.com/home",
  "redirects": 1,
  "https": true,
  "certificate": {
    "issuer": "Let's Encrypt",
    "subject": "example.com",
    "valid_from": "...",
    "valid_to": "..."
  },
  "danger_level": 1, 
  "reason": ["HTTPでもアクセス可能"]
}

※ danger_level
	•	0 = 問題なし
	•	1 = 注意
	•	2 = 危険（アクセス非推奨）

⸻

POST /qr/access-log

「アクセスする」押下時に履歴登録

{
  "url": "https://example.com",
  "danger_level": 1
}


⸻

4.3 履歴

GET /history

レスポンス：

[
  {
    "id": 123,
    "url": "https://example.com",
    "scanned_at": "2025-01-01T00:00:00Z",
    "danger_level": 0
  }, ...
]


⸻

DELETE /history/{id}

⸻

POST /report

危険URL報告

{
  "url": "https://example.com/phish"
}


⸻

5. DB設計（MySQL）

users

カラム	型	説明
id	BIGINT PK	
email	VARCHAR(255) UNIQUE	
created_at	DATETIME	
updated_at	DATETIME	


⸻

magic_links

| カラム | 型 | 説明 |
| token | VARCHAR(255) PK | ワンタイム |
| user_id | BIGINT | |
| expires_at | DATETIME | |
| used | BOOLEAN | |

⸻

scan_history

| カラム | 型 | 説明 |
| id | BIGINT PK | |
| user_id | BIGINT FK | |
| url | TEXT | |
| final_url | TEXT | |
| danger_level | INT | |
| scanned_at | DATETIME | |

⸻

bad_urls（報告された危険URL）

| カラム | 型 | 説明 |
| id | BIGINT PK |
| url | TEXT UNIQUE |
| reported_count | INT |
| first_report_at | DATETIME |

⸻

6. Docker構成

/backend
  - FastAPI
  - Dockerfile
/frontend
  - React (Vite)
  - Dockerfile
/nginx
  - reverse proxy config
/docker-compose.yml

docker-compose.yml（全体構成）
	•	nginx（80 → frontend/build、APIへproxy）
	•	backend
	•	mysql
	•	frontend（ビルド後 nginx へコピーでも可）

⸻

7. 危険判定ロジック（簡易版）

✔ HTTPS未対応 → +1

✔ 証明書期限切れ → +2

✔ 1度以上リダイレクト → +1

✔ ブラックリストに一致 → +3

✔ 国外にホストされている → +1

（※IP判定）

0〜1 → 安全
2〜3 → 注意
4〜 → 危険

⸻

8. 今後作成すべき項目
	•	画面モック（Figma）
	•	詳細API（リクエスト/レスポンス）
	•	ER図
	•	シーケンス図（Magic Link / QRスキャン）
	•	Dockerfile レイアウト
	•	ディレクトリ構成（backend/frontend）

⸻
