# CLAUDE.md - プロジェクトガイド

このドキュメントは、Claude Code や他のAIアシスタントがこのプロジェクトを理解し、効果的に作業するためのガイドです。

## 📋 プロジェクト概要

**プロジェクト名**: Visualize - 図解アプリケーション

**目的**: ユーザーが入力した日本語テキストを解析し、適切な図（フローチャート、シーケンス図、マインドマップ、ガントチャート、円グラフ）として視覚化する Webアプリケーション

**技術スタック**:
- フロントエンド: HTML5, CSS3, Vanilla JavaScript
- 図の描画: Mermaid.js v10
- デプロイ: GitHub Pages

**公開URL**: https://yama2024.github.io/visualize/

## 📁 プロジェクト構造

```
visualize/
├── index.html          # メインアプリケーション（SPA）
├── README.md          # ユーザー向けドキュメント
├── DEPLOYMENT.md      # デプロイ手順
├── CLAUDE.md          # このファイル（AI向けガイド）
└── .git/              # Gitリポジトリ
```

### ファイル詳細

**index.html** (1618行)
- 完全に自己完結型のシングルページアプリケーション
- HTML構造、CSSスタイル、JavaScriptロジックをすべて含む
- 外部依存: Mermaid.js CDNのみ
- 高精度テキスト解析エンジン搭載
- ズーム・フルスクリーン機能実装

## 🏗️ アーキテクチャと設計思想

### 設計原則

1. **ゼロ依存**: npm や build ツール不要、ブラウザで直接実行可能
2. **シンプルさ**: 単一HTMLファイルで完結
3. **オフライン対応の準備**: CDN以外の外部リソース不要
4. **レスポンシブ**: モバイル、タブレット、デスクトップ対応
5. **日本語ファースト**: 日本語の自然な入力を想定

### コンポーネント構成

```
┌─────────────────────────────────────┐
│         Header Section              │
│   (タイトル、説明)                    │
├─────────────────┬───────────────────┤
│  Input Section  │  Output Section   │
│  ┌───────────┐  │  ┌─────────────┐ │
│  │図タイプ選択│  │  │             │ │
│  ├───────────┤  │  │   Mermaid   │ │
│  │テキスト    │  │  │   Diagram   │ │
│  │入力エリア  │  │  │   Render    │ │
│  ├───────────┤  │  │   Area      │ │
│  │実行ボタン  │  │  │  ┌────────┐ │ │
│  ├───────────┤  │  │  │ Zoom   │ │ │
│  │例文リスト  │  │  │  │Controls│ │ │
│  └───────────┘  │  │  └────────┘ │ │
│                 │  └─────────────┘ │
└─────────────────┴───────────────────┘
```

## 🔧 主要機能の実装

### 1. テキスト解析エンジン

**場所**: `index.html` の `<script>` セクション

**関数**: `textToMermaid(text, type)`

**処理フロー**:
```
入力テキスト
    ↓
自動判定 or 手動選択
    ↓
キーワード検出
    ↓
専用生成関数へルーティング
    ↓
Mermaid記法生成
    ↓
レンダリング
```

**キーワード検出ロジック**:
- `containsSequenceKeywords()`: シーケンス図判定
- `containsGanttKeywords()`: ガントチャート判定
- `containsPieKeywords()`: 円グラフ判定
- `containsMindmapKeywords()`: マインドマップ判定
- デフォルト: フローチャート

### 2. 図生成関数

各図タイプに専用の生成関数があります:

#### `generateFlowchart(text)`
- 「→」「、」「。」で要素を分割
- 最初と最後を開始/終了ノードとして処理
- 「?」「判定」「チェック」を含む場合は判定ノード

#### `generateSequenceDiagram(text)`
- 参加者（アクター）を自動抽出
- 一般的な参加者名（ユーザー、システム、データベースなど）を認識
- 文章から相互作用を解析
- 「返す」「応答」などで矢印の向きを判定

#### `generateMindmap(text)`
- 階層構造を「:」「()」から抽出
- 中心トピックと子ノードを構成

#### `generateGanttChart(text)`
- 日付形式 `YYYY-MM-DD` を検出
- 「から〜まで」の期間表現を解析
- タスク名と期間をマッピング

#### `generatePieChart(text)`
- パーセント表記 `数値%` を抽出
- ラベルと値のペアを生成

### 3. セキュリティとサニタイズ

**関数**: `sanitizeMermaidText(text)`

**処理内容**:
- 特殊文字の除去（`[]{}`, `<>`, `"'`）
- 50文字制限でテキストを切り詰め
- Mermaid記法のインジェクション攻撃を防止

### 4. レンダリングシステム

**レンダリングフロー**:
```javascript
visualize() 関数
    ↓
入力検証
    ↓
Mermaid コード生成
    ↓
DOM要素作成
    ↓
mermaid.run() 実行
    ↓
エラーハンドリング
```

**特徴**:
- 非同期レンダリング（async/await）
- ユニークなダイアグラムID管理（カウンター使用）
- エラー時のユーザーフレンドリーなメッセージ表示

### 5. ズーム・フルスクリーン機能

**場所**: `index.html` の `<script>` セクション

**主要関数**:
- `initializeZoom()`: ズーム機能の初期化
- `applyZoom()`: ズーム変換の適用

**機能一覧**:

1. **ズームコントロール**
   - ズームイン/アウトボタン（20%刻み）
   - ズーム範囲: 30% ～ 300%
   - リセットボタン（100%に戻す）
   - リアルタイムズームレベル表示

2. **マウス操作**
   - **Ctrl/Cmd + マウスホイール**: スムーズズーム（10%刻み）
   - **ドラッグ＆パン**: 図の移動
   - グラブカーソルによる視覚的フィードバック

3. **フルスクリーンモード**
   - フルスクリーンボタンで画面全体表示
   - ESCキーで終了
   - 没入型閲覧体験

4. **ユーザーフィードバック**
   - トースト通知による操作確認
   - 視覚的なアニメーション

**実装詳細**:

```javascript
// グローバル状態変数
let currentZoom = 1.0;
let isDragging = false;
let startX, startY, scrollLeft, scrollTop;

// ズーム初期化
function initializeZoom() {
    currentZoom = 1.0;

    // ズームボタンのイベントリスナー
    document.getElementById('zoomInBtn').onclick = () => {
        if (currentZoom < 3.0) {
            currentZoom += 0.2;
            applyZoom();
            showToast(`ズーム: ${Math.round(currentZoom * 100)}%`);
        }
    };

    // マウスホイールズーム
    viewport.addEventListener('wheel', (e) => {
        if (e.ctrlKey || e.metaKey) {
            e.preventDefault();
            const delta = e.deltaY > 0 ? -0.1 : 0.1;
            currentZoom = Math.max(0.3, Math.min(3.0, currentZoom + delta));
            applyZoom();
        }
    }, { passive: false });

    // ドラッグ＆パン
    viewport.addEventListener('mousedown', (e) => {
        if (e.target === viewport || viewport.contains(e.target)) {
            isDragging = true;
            viewport.style.cursor = 'grabbing';
            startX = e.pageX - viewport.offsetLeft;
            startY = e.pageY - viewport.offsetTop;
            scrollLeft = viewport.scrollLeft;
            scrollTop = viewport.scrollTop;
        }
    });
}

// ズーム適用
function applyZoom() {
    const diagram = document.querySelector('#output .mermaid');
    if (diagram) {
        diagram.style.transform = `scale(${currentZoom})`;
        diagram.style.transformOrigin = 'top left';
    }
    document.getElementById('zoomLevel').textContent =
        `${Math.round(currentZoom * 100)}%`;
}
```

**CSS実装**:

```css
.zoom-controls {
    position: absolute;
    bottom: 20px;
    right: 20px;
    display: flex;
    flex-direction: column;
    gap: 8px;
    opacity: 0;
    transition: all 0.3s ease;
}

.output-area:hover .zoom-controls {
    opacity: 1;
}

.zoom-btn {
    background: white;
    border: 2px solid #667eea;
    border-radius: 8px;
    padding: 10px;
    cursor: pointer;
    transition: all 0.2s ease;
}

.output-area.fullscreen {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 9999;
    background: white;
}
```

## 🎨 UIデザイン

### カラースキーム

```css
プライマリグラデーション: #667eea → #764ba2
背景: linear-gradient(135deg, #667eea 0%, #764ba2 100%)
カード背景: white
テキスト: #333
ボーダー: #ddd
エラー: #fee (背景), #c33 (テキスト)
```

### レスポンシブブレークポイント

```css
@media (max-width: 968px) {
    /* 2カラムから1カラムへ */
    grid-template-columns: 1fr;
}
```

## 🔄 状態管理

**グローバル変数**:
- `currentDiagramType`: 選択中の図タイプ（'auto', 'flowchart', 'sequence', etc.）
- `diagramCounter`: レンダリングされた図の数（ユニークID生成用）
- `currentZoom`: 現在のズームレベル（0.3 ～ 3.0）
- `isDragging`: ドラッグ操作中かどうかのフラグ
- `startX, startY`: ドラッグ開始位置
- `scrollLeft, scrollTop`: スクロール位置の保存

## 🎯 開発ガイドライン

### 新しい図タイプの追加方法

1. **ボタンの追加**:
```html
<button class="diagram-type-btn" data-type="新しいタイプ">新しい図</button>
```

2. **検出関数の作成**:
```javascript
function containsNewTypeKeywords(text) {
    return /キーワード1|キーワード2/i.test(text);
}
```

3. **生成関数の作成**:
```javascript
function generateNewType(text) {
    let mermaidCode = '新しいMermaid記法\n';
    // テキスト解析とコード生成
    return mermaidCode;
}
```

4. **ルーティングに追加**:
```javascript
function textToMermaid(text, type) {
    if (containsNewTypeKeywords(text)) {
        return generateNewType(text);
    }
    // ...
}
```

5. **例文の追加**:
```html
<li data-example="新しいタイプ">例文テキスト</li>
```

### コーディング規約

- **インデント**: スペース4つ
- **変数名**: キャメルケース（`currentDiagramType`）
- **関数名**: キャメルケース、動詞で始める（`generateFlowchart`）
- **コメント**: 複雑なロジックには日本語コメント推奨
- **文字列**: シングルクォート使用（テンプレートリテラルを除く）

### テスト方法

**ローカルテスト**:
```bash
python -m http.server 8000
# または
npx serve
```

**テストケース**:
1. 各図タイプの例文で正しく生成されるか
2. 空入力のエラーハンドリング
3. 特殊文字を含む入力
4. 非常に長いテキスト
5. モバイルビューでのレスポンシブ動作
6. ズーム機能の動作確認（ボタン、マウスホイール）
7. フルスクリーンモードの切り替え
8. ドラッグ＆パン操作の正常動作

## 🐛 トラブルシューティング

### よくある問題

**問題1**: 図が表示されない
- **原因**: Mermaid.js CDNへのアクセス失敗
- **確認**: ブラウザコンソールでネットワークエラーをチェック
- **解決**: インターネット接続確認、CDN URLの確認

**問題2**: 日本語が文字化け
- **原因**: 文字エンコーディング設定
- **確認**: `<meta charset="UTF-8">` が設定されているか
- **解決**: ファイルをUTF-8で保存

**問題3**: レンダリングエラー
- **原因**: Mermaid記法の構文エラー
- **確認**: コンソールログで生成されたMermaidコードを確認
- **解決**: `sanitizeMermaidText()` の改善、生成ロジックの修正

**問題4**: 図が重複表示される
- **原因**: `diagramCounter` の管理ミス
- **確認**: DOM内のダイアグラムID
- **解決**: カウンターのリセット、ユニークID生成の確認

**問題5**: ズーム操作が反応しない
- **原因**: イベントリスナーの初期化ミス、または図がレンダリングされていない
- **確認**: コンソールでエラーを確認、`initializeZoom()` が呼ばれているか
- **解決**: 図のレンダリング後にズーム初期化を実行

**問題6**: フルスクリーンから戻れない
- **原因**: ESCキーイベントリスナーの設定ミス
- **確認**: フルスクリーンボタンのクリックイベント確認
- **解決**: ESCキーハンドラーの追加、ボタンの状態管理確認

**問題7**: ドラッグ操作が重い
- **原因**: イベントハンドラーで過剰な処理
- **確認**: パフォーマンスプロファイラーで確認
- **解決**: throttle/debounce の実装、CSS transform の活用

## 🚀 パフォーマンス最適化

### 現在の最適化

1. **CDN使用**: Mermaid.jsをCDN経由で読み込み（初回キャッシュ後高速）
2. **インライン化**: CSS/JSを単一ファイルに統合（HTTPリクエスト削減）
3. **遅延初期化**: `startOnLoad: false` でMermaid初期化を制御
4. **CSS Transform**: ズーム機能でGPUアクセラレーションを活用
5. **イベント最適化**: `passive: false` で必要な場合のみデフォルト動作を防止

### 将来的な改善案

1. **Service Worker**: オフライン対応
2. **Mermaidローカルコピー**: CDN依存を排除
3. **履歴機能**: LocalStorageで過去の図を保存
4. **テーマ切り替え**: ダークモード対応
5. **ズーム位置の記憶**: ユーザーのズームレベルとパン位置をLocalStorageに保存
6. **タッチジェスチャー**: ピンチズーム、スワイプパンのサポート（モバイル最適化）

## 📊 使用されているMermaid記法

### サポート図タイプ

1. **Flowchart** (`graph TD`)
   - ノード形状: `[]`, `()`, `{}`
   - 矢印: `-->`

2. **Sequence Diagram** (`sequenceDiagram`)
   - 参加者: `participant`
   - メッセージ: `->`, `-->`

3. **Mindmap** (`mindmap`)
   - ルート: `root((text))`
   - 子ノード: インデント

4. **Gantt Chart** (`gantt`)
   - 日付形式: `YYYY-MM-DD`
   - 期間: 開始日, 終了日

5. **Pie Chart** (`pie`)
   - データ: `"ラベル" : 値`

## 🔐 セキュリティ考慮事項

### 実装済み対策

1. **XSS防止**: `sanitizeMermaidText()` で特殊文字除去
2. **インジェクション防止**: Mermaid記法の制御文字を除去
3. **サイズ制限**: テキスト長を50文字に制限

### 追加推奨対策

1. CSP (Content Security Policy) ヘッダー設定
2. 入力検証の強化
3. レート制限（連続実行の制御）

## 📝 変更履歴

### v1.1.0 (2025-11-18)
- 🔍 ズーム・フルスクリーン機能の追加
  - ズームイン/アウトボタン（30%～300%）
  - Ctrl/Cmd + マウスホイールズーム
  - ドラッグ＆パン機能
  - フルスクリーンモード（ESCキーで終了）
  - リアルタイムズームレベル表示
- 🎯 高精度テキスト解析エンジンの実装
  - スコアリングベースの図タイプ自動判定
  - 日本語助詞・動詞パターン認識の強化
  - キーワード重み付けシステム（Primary: 10, Secondary: 5, Tertiary: 2）
- ✨ UI/UXの最適化
  - アニメーション強化
  - トースト通知システム
  - コピー・ダウンロード機能

### v1.0.0 (2025-11-17)
- 初回リリース
- 5種類の図タイプをサポート
- 日本語自動判定機能
- レスポンシブUI実装
- GitHub Pages対応

## 🤝 コントリビューションガイド

### Pull Request作成時の確認事項

- [ ] すべての図タイプで動作確認
- [ ] ブラウザコンソールにエラーがないか
- [ ] モバイルビューで表示確認
- [ ] ズーム・フルスクリーン機能の動作確認
- [ ] README.mdの更新（機能追加時）
- [ ] CLAUDE.mdの更新（機能追加時）
- [ ] 例文の追加（新機能追加時）

### コミットメッセージ規約

```
<type>: <subject>

<body>
```

**Type**:
- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメント更新
- `style`: コードスタイル修正
- `refactor`: リファクタリング
- `test`: テスト追加
- `chore`: ビルド、設定変更

## 🎓 学習リソース

### Mermaid.js
- 公式ドキュメント: https://mermaid.js.org/
- ライブエディタ: https://mermaid.live/

### 参考情報
- 正規表現テスト: https://regex101.com/
- CSS Grid: https://css-tricks.com/snippets/css/complete-guide-grid/
- Flexbox: https://css-tricks.com/snippets/css/a-guide-to-flexbox/

## 🎯 今後の開発ロードマップ

### Phase 2（実装済み） ✅
- [x] ズーム・フルスクリーン機能
- [x] 高精度テキスト解析エンジン
- [x] トースト通知システム

### Phase 3（予定）
- [ ] エクスポート機能（SVG, PNG）
- [ ] 履歴機能（LocalStorage）
- [ ] テーマ切り替え（ライト/ダーク）
- [ ] より高度なテキスト解析（NLP/AI統合）
- [ ] タッチジェスチャー対応（ピンチズーム）

### Phase 4（予定）
- [ ] ユーザーアカウント機能
- [ ] クラウド保存
- [ ] 共有機能（URL生成）
- [ ] リアルタイムコラボレーション
- [ ] カスタムテーマエディター

---

**最終更新**: 2025年11月18日
**メンテナ**: Claude Code
**ライセンス**: MIT License
