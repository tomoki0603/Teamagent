---
name: aws-eol-manager
description: AWSサービスのEOL(End of Life)対応に必要な調査・管理資料をExcelで生成するスキル。管理用の全体俯瞰資料と、サービスごとの個別調査資料を分けて出力する。「AWS EOLを調べて」「EOL管理表を作って」「AWSのEOL対応して」「aws-eol-managerを実行して」「EOLを確認して」と言ったときに使用。
metadata:
  version: 1.1.0
  category: documentation
---

# AWS EOL Manager — EOL対応 調査・管理資料生成スキル

## 概要

複数のAWSアカウントにおけるEOL対象リソースについて、Web上の公開情報をもとに調査を行い、
以下の2種類のExcel資料を生成する：

1. **管理用資料**（全サービス横断の一覧・タスク・スケジュール）
2. **個別サービス資料**（サービスごとの調査内容・EOL手順・期限・参考文献）

参照ファイル構成:
- `references/01-excel-templates.md` — Excelシート構成・カラム定義
- `references/02-eol-research-guide.md` — EOL調査ガイド・優先度判定基準
- `references/03-cloudshell-discovery.md` — CloudShellリソース洗い出しガイド

### 制約事項
- このPCから会社のAWSアカウントへの直接操作は一切行わない
- Web検索による公開情報の調査と、管理用資料の作成のみを行う
- AWS公式ドキュメント・ブログ・アナウンスを情報源とする

---

## CRITICAL: 一問一答ルール

**必ず1つの質問だけを行い、回答を待ってから次に進む。**

---

## Step 1: 対象サービスのヒアリング

ユーザーに以下を確認する:

```
EOL調査の対象となるAWSサービス・リソースを教えてください。

以下の情報があると精度が上がります：
・対象AWSアカウント名（複数ある場合はすべて）
・EOLが気になるサービス名（例：Amazon Linux 2、RDS MySQL 5.7、EKS 1.27 など）
・現在使用中のバージョン（わかる範囲で）

例：
  アカウント: production-account, staging-account
  サービス: Amazon Linux 2, RDS MySQL 5.7, ElastiCache Redis 6.x, EKS 1.27
```

ユーザーの回答を受けて、調査対象リストを作成する。

---

## Step 2: EOL情報のWeb調査

各サービスについて WebSearch を使い、以下の情報を調査する。
詳細な調査項目は `references/02-eol-research-guide.md` を参照。

### 調査項目（サービスごと）
1. **EOL日程**: AWS公式のEOL・EOS（End of Support）日程
2. **延長サポート**: 延長サポートの有無・期間・追加費用
3. **移行先**: AWS推奨の移行先バージョン/サービス
4. **移行手順**: 公式ドキュメントに記載の移行方法・手順
5. **影響範囲**: EOLによる影響（セキュリティパッチ停止・機能制限等）
6. **参考文献URL**: AWS公式ドキュメント・ブログ・アナウンスのURL

### 検索クエリ例
```
"[サービス名] end of life AWS"
"[サービス名] EOL date AWS"
"[サービス名] migration guide AWS"
"[サービス名] end of support AWS 2025 2026"
```

**CRITICAL: 各サービスの調査時に参考文献のURLを必ず収集・記録すること。**

---

## Step 3: 個別サービス資料の生成（サービスごとに1ファイル）

各サービスについて、Python（openpyxl）でExcelファイルを生成する。
シート構成・カラム定義は `references/01-excel-templates.md` の「個別サービス資料」セクションを参照。

### 出力ファイル
`[出力先]/[サービス名]-eol-report-YYYY-MM-DD.xlsx`

### シート構成
- **Sheet1: 調査内容まとめ** — サービス概要・影響範囲・現状整理
- **Sheet2: EOL手順** — 移行ステップ・作業手順・注意事項
- **Sheet3: EOL期限** — 公式期限・延長サポート期限・推奨対応期限
- **Sheet4: 参考文献** — AWS公式ドキュメント・ブログ等のリンク集
- **Sheet5: CloudShellリソース洗い出し** — 対象リソースを洗い出すCloudShellコマンド集

---

## Step 4: 管理用資料の生成（全サービス横断）

全サービスの情報を統合し、Python（openpyxl）で管理用Excelファイルを生成する。
シート構成・カラム定義は `references/01-excel-templates.md` の「管理用資料」セクションを参照。

### 出力ファイル
`[出力先]/aws-eol-management-YYYY-MM-DD.xlsx`

### シート構成
- **Sheet1: EOL対象サービス一覧** — 全サービスの横断一覧
- **Sheet2: 対応タスク一覧** — 担当・期限・ステータス管理
- **Sheet3: スケジュール** — タイムライン・マイルストーン
- **Sheet4: CloudShellリソース洗い出し** — 全サービスのCloudShellコマンド集

---

## Step 5: ファイル保存と完了通知

### 保存処理

**CRITICAL: 必ず以下の手順でファイルを保存すること。チャット表示だけで終わらせてはいけない。**

1. **openpyxl のインストール確認:**
   ```bash
   pip install openpyxl 2>/dev/null || pip3 install openpyxl 2>/dev/null
   ```

2. **Bash ツールでディレクトリを作成する:**
   ```bash
   mkdir -p [プロジェクトルート]/aws-eol-reports/YYYY-MM-DD
   ```

3. **Bash ツールでPythonスクリプトを実行し、Excelファイルを生成する:**
   - 個別サービス資料: `[サービス名]-eol-report-YYYY-MM-DD.xlsx` × サービス数
   - 管理用資料: `aws-eol-management-YYYY-MM-DD.xlsx` × 1

4. **生成されたファイルの存在確認:**
   ```bash
   ls -la [出力先]/
   ```

### チャット出力

**CRITICAL: 調査内容の全文はチャットには表示しない。完了通知のみ表示する。**

保存完了後に以下を表示:
```
✅ AWS EOL調査資料 生成完了

📁 保存先: aws-eol-reports/YYYY-MM-DD/

📊 管理用資料（全体俯瞰）:
  aws-eol-management-YYYY-MM-DD.xlsx

📋 個別サービス資料:
  [サービス名1]-eol-report-YYYY-MM-DD.xlsx
  [サービス名2]-eol-report-YYYY-MM-DD.xlsx
  ...

📄 調査対象: N件
🕐 生成日時: YYYY-MM-DD HH:MM

💬 内容を修正・追加したい場合は「EOL資料を更新して」と入力してください。
```

---

## エラーハンドリング

### Pythonまたはopenpyxlが利用できない場合
```
「Excel生成に必要なPython環境が見つかりません。
以下のコマンドでopenpyxlをインストールしてから再実行してください：

pip install openpyxl

インストール後に「続けて」と入力してください。」
```

### Web検索でEOL情報が見つからない場合
該当サービスの個別資料に「公式情報未確認」と明記し、処理を続行する。
完了通知に以下を追加:
```
「⚠️ 以下のサービスはEOL情報が確認できませんでした:
・[サービス名]
AWS公式サイトで最新情報を直接確認してください。」
```

### サービス名が不明確な場合
```
「サービス名をもう少し具体的に教えてください。例えば：
・「RDS」→ 「RDS MySQL 5.7」「RDS PostgreSQL 11」
・「Lambda」→ 「Lambda Python 3.8 ランタイム」
・「EKS」→ 「EKS 1.27」
バージョンまで指定するとより正確な調査ができます。」
```
