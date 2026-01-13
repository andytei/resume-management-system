# 履歴書管理システム / Resume Management System

AI搭載の日本語履歴書管理システム。複数のファイル形式から自動で情報を抽出し、標準化されたExcel/PDFフォーマットで出力します。

## ✨ 主要機能

### 📥 マルチフォーマット対応
- **Excel** (.xlsx, .xls)
- **PDF** (.pdf)  
- **Word** (.doc, .docx)
- **画像** (.jpg, .png) - OCR自動認識
- **音声** (.mp3, .wav) - 音声→テキスト変換

### 🤖 AI自動抽出
- Ollama (qwen2.5) による高精度な情報抽出
- 日本語に最適化されたプロンプト
- 画像からの直接抽出（Vision モデル）

### ✏️ 人工レビュー
- Webベースの直感的な編集画面
- テンプレート形式での表示
- リアルタイムバリデーション

### 📤 柔軟なエクスポート
- Excel形式
- PDF形式
- 単一/複数候補者の一括出力

## 🚀 クイックスタート

### 前提条件
```bash
# Docker と Docker Compose がインストールされていること
docker --version
docker-compose --version
```

### 起動方法

1. **リポジトリをクローン**
```bash
git clone https://github.com/your-repo/resume-management-system.git
cd resume-management-system
```

2. **環境変数設定（オプション）**
```bash
cp .env.example .env
# 必要に応じて .env を編集
```

3. **Docker起動**
```bash
# すべてのサービスを起動
docker-compose up -d

# ログを確認
docker-compose logs -f

# Ollamaモデルのダウンロード完了を待つ（初回のみ5-10分）
docker-compose logs ollama | grep "pull complete"
```

4. **アクセス**
- **フロントエンド**: http://localhost:3000
- **バックエンドAPI**: http://localhost:8000
- **API ドキュメント**: http://localhost:8000/docs

### 初回セットアップ確認

```bash
# データベース接続確認
docker-compose exec backend python -c "from app.database import engine; print('DB OK' if engine else 'DB Error')"

# Ollama接続確認
curl http://localhost:11434/api/tags

# フロントエンド確認
curl http://localhost:3000
```

## 📖 使い方

### 1. テンプレート登録
1. サイドメニューから「テンプレート管理」→「テンプレートアップロード」
2. Excelテンプレートファイルをアップロード
3. フィールドマッピングを確認

### 2. 履歴書取込
1. 「履歴書取込」→「ファイルアップロード」
2. 履歴書ファイルをドラッグ&ドロップ
3. AI抽出完了を待つ（1-2分）
4. 抽出結果を確認・修正
5. 保存

### 3. 候補者管理
1. 「候補者管理」→「候補者一覧」
2. 候補者を検索・フィルタリング
3. 詳細表示や編集が可能

### 4. エクスポート
1. 「エクスポート」ページ
2. 候補者を選択
3. テンプレートと出力形式を選択
4. エクスポート実行

## 🏗️ アーキテクチャ

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   React     │────→│   FastAPI    │────→│  PostgreSQL │
│  Frontend   │←────│   Backend    │←────│  Database   │
└─────────────┘     └──────────────┘     └─────────────┘
                           │
                           ↓
                    ┌──────────────┐
                    │    Ollama    │
                    │  (AI Model)  │
                    └──────────────┘
```

## 🛠️ 開発環境セットアップ

### バックエンド単体起動
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# データベース準備
export DATABASE_URL="postgresql://resume_user:resume_pass123@localhost:5432/resume_db"

# 起動
uvicorn app.main:app --reload --port 8000
```

### フロントエンド単体起動
```bash
cd frontend
npm install
npm start
```

### Ollama単体起動（ローカル）
```bash
# Ollama インストール
curl -fsSL https://ollama.com/install.sh | sh

# モデルダウンロード
ollama pull qwen2.5:7b
ollama pull llava:7b

# サーバー起動
ollama serve
```

## 🔧 設定

### 環境変数一覧

| 変数名 | デフォルト値 | 説明 |
|--------|-------------|------|
| DATABASE_URL | postgresql://resume_user:resume_pass123@postgres:5432/resume_db | PostgreSQL接続URL |
| OLLAMA_HOST | http://ollama:11434 | Ollama API エンドポイント |
| WHISPER_MODEL | base | Whisper音声認識モデル |
| UPLOAD_DIR | /app/uploads | アップロードファイル保存先 |
| EXPORT_DIR | /app/exports | エクスポートファイル保存先 |
| REACT_APP_API_URL | http://localhost:8000 | バックエンドAPI URL |

### モデル選択

Ollamaモデルは用途に応じて変更可能：

```yaml
# docker-compose.yml内
command:
  - |
    ollama pull qwen2.5:14b  # より高精度（要メモリ）
    # ollama pull qwen2.5:7b   # バランス型（推奨）
    # ollama pull llama3.1:8b  # 代替モデル
```

## 📊 データベーススキーマ

主要テーブル：
- `candidates` - 候補者基本情報
- `education` - 学歴
- `project_history` - プロジェクト経歴
- `technical_skills` - 技術スキル
- `templates` - テンプレート定義
- `original_resumes` - 原始ファイル

詳細は `backend/init.sql` を参照

## 🧪 テスト

```bash
# バックエンドテスト
cd backend
pytest

# フロントエンドテスト
cd frontend
npm test
```

## 📦 本番環境デプロイ

### Docker Swarmでのデプロイ
```bash
docker stack deploy -c docker-compose.yml resume-system
```

### Kubernetesでのデプロイ
```bash
# Helm チャート準備中
kubectl apply -f k8s/
```

## 🐛 トラブルシューティング

### Ollamaモデルがダウンロードされない
```bash
# 手動でモデルをダウンロード
docker-compose exec ollama ollama pull qwen2.5:7b
docker-compose exec ollama ollama pull llava:7b
```

### データベース接続エラー
```bash
# データベースコンテナの状態確認
docker-compose ps postgres
docker-compose logs postgres

# 再起動
docker-compose restart postgres
```

### フロントエンドが起動しない
```bash
# node_modules再インストール
docker-compose exec frontend rm -rf node_modules
docker-compose exec frontend npm install
docker-compose restart frontend
```

## 🤝 コントリビューション

プルリクエストを歓迎します！

1. Fork する
2. Feature ブランチを作成 (`git checkout -b feature/AmazingFeature`)
3. 変更をコミット (`git commit -m 'Add some AmazingFeature'`)
4. ブランチにプッシュ (`git push origin feature/AmazingFeature`)
5. プルリクエストを作成

## 📝 ライセンス

MIT License - 詳細は `LICENSE` ファイルを参照

## 👨‍💻 作者

あなたの名前

## 🙏 謝辞

- [Ollama](https://ollama.com/) - ローカルLLMランタイム
- [FastAPI](https://fastapi.tiangolo.com/) - 高性能Pythonフレームワーク
- [Ant Design](https://ant.design/) - React UIライブラリ
- [OpenAI Whisper](https://github.com/openai/whisper) - 音声認識モデル