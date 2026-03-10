# 01 — YAML フロントマター仕様

## 基本構造

```yaml
---
name: skill-name
description: スキルの説明とトリガーワード。「〇〇して」「〇〇を実行して」と言ったときに使用。
metadata:
  version: 1.0.0
  mcp-server: playwright   # 不要な場合は省略
  category: カテゴリ名
---
```

---

## 各フィールドの詳細

### `name`
- **形式**: kebab-case（小文字・ハイフン区切り）
- **例**: `nikkei-briefing`, `code-reviewer`, `github-issue-tracker`
- **ルール**: `.claude/skills/` 配下のフォルダ名と完全に一致させる

### `description`
- Claudeがどのスキルを起動すべきかを判断する **最重要フィールド**
- 日本語でトリガーワードを具体的に列挙する
- フォーマット: `[何をするスキルか]。「[trigger1]」「[trigger2]」と言ったときに使用。`
- 詳細は `03-description-guide.md` を参照

### `metadata.version`
- セマンティックバージョニング: `1.0.0`
- 大きな変更: メジャー番号を上げる（2.0.0）
- 機能追加: マイナー番号を上げる（1.1.0）
- バグ修正: パッチ番号を上げる（1.0.1）

### `metadata.mcp-server`
使用するMCPサーバー名を記載する。不要な場合は省略。

| MCPサーバー | 用途 |
|------------|------|
| `playwright` | ブラウザ操作・Webスクレイピング |
| `github` | GitHub Issue・PR・リポジトリ操作 |
| `notion` | Notionページ・データベース操作 |
| `postgresql` | PostgreSQLデータベースクエリ |
| `slack` | Slackメッセージ送受信 |

複数のMCPが必要な場合:
```yaml
metadata:
  mcp-server:
    - playwright
    - github
```

### `metadata.category`
スキルの分類名（自由記述）:
- `investment-research` — 投資・金融リサーチ
- `code-review` — コードレビュー・品質管理
- `automation` — 定期実行・自動化
- `documentation` — ドキュメント生成
- `data-analysis` — データ分析・集計
- `meta-skill` — スキルを作るスキル

---

## よくある間違い

❌ **フロントマターの前後に余白がない**
```yaml
---
name: my-skill
---
# スキル名  ← この前に空行が必要
```

✅ **正しい形式**
```yaml
---
name: my-skill
---

# スキル名
```

❌ **nameがフォルダ名と一致していない**
```
フォルダ: .claude/skills/nikkei-briefing/
name: nikkei_briefing  ← アンダースコアはNG
```

✅ **一致している**
```
フォルダ: .claude/skills/nikkei-briefing/
name: nikkei-briefing
```
