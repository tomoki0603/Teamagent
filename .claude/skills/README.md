# スキル一覧

このディレクトリ配下のスキルは Claude Code から自動検出される。

## 登録済みスキル

| スキル名 | カテゴリ | バージョン | MCP | 概要 |
|---------|---------|-----------|-----|------|
| skill-creator-max | meta-skill | 1.0.0 | なし | 対話形式でAgent Skillsを設計・生成するマスターツール |
| aws-eol-manager | documentation | 1.1.0 | なし | AWS EOL対応の調査・管理資料をExcelで生成 |
| jp-us-stock-ranking | data-collection | 1.0.0 | なし | 日本株・米国株の市場ランキング（値上がり/値下がり/出来高/売買代金）をExcelで保存。チームエージェント使用 |
| kindle-obsidian-summary | documentation | 1.3.0 | なし | Kindle書籍のPDF/スクリーンショットから章別要約を生成しObsidianに保存。Kindle for PC + pyautogui自動キャプチャ対応 |

## スキル管理手順

### 新規追加
1. `skill-creator-max` を使用（「新しいスキルを作りたい」と入力）
2. 対話に従いスキルが `.claude/skills/[name]/` に生成される
3. **この README.md の一覧表にスキルを追記する**
4. `git add .claude/skills/[name]/ .claude/skills/README.md`
5. `git commit -m "Add skill: [name]"`

### 更新
1. 該当スキルの `SKILL.md` や `references/` を編集
2. `SKILL.md` 内の `metadata.version` を更新（セマンティックバージョニング）
3. この README.md の一覧表のバージョンも更新
4. `git commit -m "Update skill: [name] to v[version]"`

### 削除
1. `.claude/skills/[name]/` ディレクトリを削除
2. この README.md の一覧表から該当行を削除
3. `git add -A && git commit -m "Remove skill: [name]"`

## ディレクトリ構造

```
.claude/skills/
├── README.md                     ← このファイル（スキル一覧）
└── [skill-name]/                 ← 各スキルのフォルダ
    ├── SKILL.md                  ← スキル定義（必須）
    └── references/               ← 参照ドキュメント（任意）
        ├── 01-[topic].md
        └── 02-[topic].md
```

## 命名規則
- フォルダ名: **kebab-case**（小文字・ハイフン区切り）
- `SKILL.md` 内の `name` フィールドはフォルダ名と完全一致させる
- references ファイル: `[番号]-[内容].md`（例: `01-categories.md`）
