# 🚀 デプロイメントガイド

## GitHub Pagesへのデプロイ手順

このアプリケーションをGitHub Pagesで公開するための手順です。

### 1. プルリクエストの作成

現在のブランチ `claude/visualization-diagram-app-01UGWEB5KUpjaBKfc8h1rNqo` をメインブランチにマージするため、プルリクエストを作成します。

#### GitHub Web UIでの手順:

1. GitHubリポジトリ（https://github.com/yama2024/visualize）にアクセス
2. 「Pull requests」タブをクリック
3. 「New pull request」ボタンをクリック
4. Base: `main`（または既定のブランチ）を選択
5. Compare: `claude/visualization-diagram-app-01UGWEB5KUpjaBKfc8h1rNqo` を選択
6. 「Create pull request」をクリック
7. タイトルと説明を入力:
   - **タイトル**: `Add visualization diagram application`
   - **説明**:
     ```
     ## 概要
     ユーザー入力を図解するビジュアライゼーションアプリケーションを追加

     ## 機能
     - フローチャート、シーケンス図、マインドマップ、ガントチャート、円グラフに対応
     - 日本語テキストからの自動図解
     - Mermaid.jsによる高品質なレンダリング
     - レスポンシブデザイン

     ## デプロイ
     - GitHub Pages対応
     - 静的HTMLアプリケーション
     ```
8. 「Create pull request」をクリック
9. レビュー後、「Merge pull request」をクリック
10. 「Confirm merge」をクリック

### 2. GitHub Pagesの有効化（シンプルな方法）

マージ完了後、GitHub Pagesを設定します。

#### ブランチベースのデプロイ（推奨）:

1. GitHubリポジトリページで「Settings」タブをクリック
2. 左サイドバーの「Pages」をクリック
3. **Source** セクションで:
   - Source: `Deploy from a branch` を選択
   - Branch: `main` を選択
   - Folder: `/ (root)` を選択
4. 「Save」をクリック

数分後、`https://yama2024.github.io/visualize/` でアプリケーションが公開されます。

### 3. GitHub Actionsでの自動デプロイ（高度な方法）

より細かい制御が必要な場合、GitHub Actionsを使用できます。

#### 手順:

1. リポジトリに `.github/workflows/deploy.yml` ファイルを作成:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

2. Settings → Pages → Source で `GitHub Actions` を選択

3. mainブランチにプッシュすると自動デプロイされます

### 4. デプロイ確認

1. Settings → Pages で公開URLを確認
2. 以下のURLでアクセス可能:
   ```
   https://yama2024.github.io/visualize/
   ```
3. GitHub Actionsを使用した場合は、「Actions」タブで進行状況を確認

## カスタムドメインの設定（オプション）

1. Settings → Pages → Custom domain
2. ドメイン名を入力（例: `visualize.example.com`）
3. DNSレコードを設定:
   ```
   Type: CNAME
   Name: visualize
   Value: yama2024.github.io
   ```
4. 「Enforce HTTPS」にチェック

## トラブルシューティング

### デプロイが失敗する場合

1. **Permissions エラー**:
   - Settings → Actions → General → Workflow permissions
   - 「Read and write permissions」を選択
   - 「Allow GitHub Actions to create and approve pull requests」にチェック

2. **Pages が表示されない**:
   - Settings → Pages で Source が正しく設定されているか確認
   - Actions タブでワークフローが成功しているか確認
   - ブラウザのキャッシュをクリア

3. **404 エラー**:
   - `index.html` がリポジトリのルートにあるか確認
   - 数分待ってから再度アクセス（デプロイには時間がかかる場合があります）

## ローカルでのテスト

デプロイ前にローカルでテスト:

```bash
# Python
python -m http.server 8000

# Node.js
npx serve

# PHP
php -S localhost:8000
```

ブラウザで `http://localhost:8000` にアクセス

## デプロイURL

本番環境: https://yama2024.github.io/visualize/

---

更新日: 2025年11月17日
