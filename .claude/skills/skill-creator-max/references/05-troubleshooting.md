# 05 — トラブルシューティング

## 問題カテゴリ一覧

1. [スキルが起動しない](#スキルが起動しない)
2. [MCPサーバーの問題](#MCPサーバーの問題)
3. [ファイル操作の問題](#ファイル操作の問題)
4. [Web操作の問題](#Web操作の問題)
5. [出力品質の問題](#出力品質の問題)

---

## スキルが起動しない

### 症状
「[トリガーワード]」と入力してもスキルが起動せず、Claudeが通常の会話として返答する。

### 原因と対処

**1. descriptionのトリガーワードが合っていない**
```yaml
# 問題: ユーザーの言葉と一致しない
description: 「ニュース収集を開始してください」と言ったときに使用。

# 解決: 自然な言葉に修正
description: 「日経をまとめて」「ニュースをまとめて」「朝刊チェックして」と言ったときに使用。
```

**2. YAMLフロントマターの書式エラー**
```yaml
# 問題: --- が欠けている
name: my-skill
description: ...

# 解決: --- で囲む
---
name: my-skill
description: ...
---
```

**3. フォルダ名とnameフィールドが一致していない**
```
フォルダ: .claude/skills/my-skill/
name: my_skill  ← アンダースコアはNG

解決:
name: my-skill  ← ハイフンで統一
```

**4. SKILL.mdがフォルダ直下にない**
```
❌ .claude/skills/my-skill/docs/SKILL.md
✅ .claude/skills/my-skill/SKILL.md
```

---

## MCPサーバーの問題

### Playwright MCPが動かない

#### 症状
`browser_navigate` などのツールが見つからない、またはエラーになる。

#### 設定ファイルの確認
`.mcp.json` が正しく設定されているか確認する:

**Windows の場合:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "cmd",
      "args": [
        "/c", "npx", "@playwright/mcp@latest",
        "--user-data-dir=C:/Users/[username]/.playwright-profile",
        "--browser=chromium"
      ]
    }
  }
}
```
⚠️ **Windows では `cmd` + `/c` のラッパーが必要。** `npx` を直接 command に指定すると動かない。

**Mac / Linux の場合:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--user-data-dir=/Users/[username]/.playwright-profile",
        "--browser=chromium"
      ]
    }
  }
}
```

#### よくある原因
| 原因 | 対処 |
|------|------|
| Windowsで`npx`を直接指定 | `cmd` + `/c` に変更 |
| `@playwright/mcp@latest` が未インストール | `npx @playwright/mcp@latest` を一度実行 |
| `--user-data-dir` のパスが存在しない | フォルダを手動作成するか、パスを修正 |
| Claude Codeを再起動していない | `.mcp.json` 変更後は要再起動 |

---

### GitHub MCPが動かない

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "[your-token]"
      }
    }
  }
}
```

⚠️ `GITHUB_PERSONAL_ACCESS_TOKEN` の設定が必要。
GitHubの Settings → Developer settings → Personal access tokens で発行する。

---

### Notion MCPが動かない

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer [notion-token]\", \"Notion-Version\": \"2022-06-28\"}"
      }
    }
  }
}
```

---

## ファイル操作の問題

### ファイルが保存されない

**症状**: スキルが完了してもファイルが生成されない。

**原因**: SKILL.mdにWriteツールの明示的な指示がない。

**対処**: SKILL.mdに以下を追加する:
```markdown
**CRITICAL: 必ずWriteツールでファイルを保存すること。**
チャット表示だけで終わらせてはいけない。

Writeツールで以下のパスに保存する:
`C:/Users/[username]/Desktop/[project]/output/YYYY-MM/filename-YYYY-MM-DD.md`
```

---

### ディレクトリが存在しないエラー

**症状**: `YYYY-MM` 形式のフォルダが新月に存在せず保存に失敗する。

**対処**: SKILL.mdに以下を追加する:
```markdown
Bashツールでディレクトリを作成する（存在しない場合）:
```bash
mkdir -p C:/Users/[username]/Desktop/[project]/output/YYYY-MM
```
その後Writeツールでファイルを保存する。
```

---

### パスのスラッシュ問題（Windows）

**症状**: Windows でバックスラッシュ（`\`）を使うとエラーになる場合がある。

**対処**: SKILL.mdのパス指定は常にフォワードスラッシュ（`/`）を使う:
```
✅ C:/Users/start/Desktop/Teamagent/output/
❌ C:\Users\start\Desktop\Teamagent\output\
```

---

## Web操作の問題

### ログインが必要なページにアクセスできない

**症状**: ログインページにリダイレクトされる。

**対処**:
1. `--user-data-dir` に永続プロファイルを設定する（.mcp.jsonの設定参照）
2. 一度 Playwright ブラウザで手動ログインしてセッションを保存する
3. 次回以降は保存済みセッションを使用してアクセスする

SKILL.mdに以下を明記する:
```markdown
**前提条件**: ログイン済みのPlaywrightセッションが必要。
初回は手動でブラウザを開きログインしてからスキルを実行すること。
```

---

### ページ要素が取得できない

**症状**: `browser_snapshot` でターゲット要素が見つからない。

**原因と対処**:
| 原因 | 対処 |
|------|------|
| サイト構造の変更 | スナップショットで実際の構造を確認し、セレクタを更新 |
| 動的ロード（JS） | `browser_wait_for` でロード完了を待つ |
| ページャー（次ページ） | スクロールまたはページネーション操作を追加 |
| ログイン切れ | セッションを更新してから再実行 |

---

## 出力品質の問題

### 出力フォーマットが乱れる

**症状**: 生成されたMarkdownの見出し・表・箇条書きが崩れる。

**対処**: SKILL.mdにフォーマットテンプレートを明示する:
```markdown
### 出力フォーマット（必ずこの形式で出力すること）

## [カテゴリ名] — [記事タイトル]

| 項目 | 内容 |
|------|------|
| 日時 | YYYY-MM-DD |
| 概要 | [1〜2文] |

[詳細説明]
```

---

### 参照ファイルの内容が反映されない

**症状**: `references/01-categories.md を参照` と書いたのに、分類が正しく行われない。

**対処**:
1. SKILL.mdの参照記述が正確か確認する:
   ```markdown
   詳細な分類基準は `references/01-categories.md` を参照。
   ```
2. 参照ファイルが実際に存在するか確認する
3. 参照ファイルの内容が具体的・明確か確認する（曖昧な記述は無視される）

---

## 設定ファイルのテンプレート集

### .mcp.json — Playwright（Windows）
```json
{
  "mcpServers": {
    "playwright": {
      "command": "cmd",
      "args": [
        "/c", "npx", "@playwright/mcp@latest",
        "--user-data-dir=C:/Users/start/.playwright-profile",
        "--browser=chromium"
      ]
    }
  }
}
```

### .mcp.json — 複数MCP同時設定
```json
{
  "mcpServers": {
    "playwright": {
      "command": "cmd",
      "args": ["/c", "npx", "@playwright/mcp@latest", "--browser=chromium"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    }
  }
}
```
