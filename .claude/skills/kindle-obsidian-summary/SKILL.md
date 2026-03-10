---
name: kindle-obsidian-summary
description: Kindleで購入した本のPDF（またはスクリーンショット画像）を読み込み、章別要約・主要な学び・アクションアイテムを生成してObsidianに保存するスキル。「Kindleの本を要約して」「本を要約して」「読書メモを作成して」「kindle-obsidian-summaryを使って」「[タイトル]を要約してObsidianに保存して」と言ったときに使用。
metadata:
  version: 1.3.0
  category: documentation
---

# Kindle × Obsidian 読書要約スキル

Kindleで購入した書籍のPDFまたはスクリーンショット画像を入力として受け取り、
章別要約・主要な学び・アクションアイテムを生成してObsidianのVaultに保存する。

著作権・利用制限については `references/02-copyright-notice.md` を参照。

---

## 定数

```
VAULT_PATH = C:/Users/start/Desktop/Obsidian/Vault_Book/Books
TEMPLATES_DIR = ${VAULT_PATH}/01_Templates
HIGHLIGHT_DIR = ${VAULT_PATH}/02_Bookhighlight
BOOKALL_DIR = ${VAULT_PATH}/03_Bookall
BOOKMEMO_DIR = ${VAULT_PATH}/04_Bookmemo
```

---

## Step 1: 書籍タイトルの確認

ユーザーに書籍タイトルを確認する：

```
要約する本のタイトルを教えてください。
（例：「ゾーン」「達成の科学」など）
```

---

## Step 2: 入力ファイルの確認と入手

ユーザーにPDFまたはスクリーンショットの有無を確認する：

```
「[書籍タイトル]」のPDFまたはスクリーンショット画像はお持ちですか？

A. PDFを持っている → ファイルパスを教えてください
B. スクリーンショット画像を持っている → 画像フォルダのパスを教えてください
C. どちらも持っていない → PDF入手方法をご案内します
```

### A / B の場合
入力形式を判定する：
- `.pdf` → PDF モードで処理（Step 3 へ）
- `.png` / `.jpg` / `.jpeg` またはフォルダパス → 画像モードで処理（Step 3 へ）

### C の場合（PDFもスクリーンショットもない）
PDF入手方法をガイドする。詳細は `references/03-pdf-acquisition-guide.md` を参照。

```
📖 PDF入手方法をご案内します。

【方法1】PDF付属の書籍の場合
  Kindle購入時にPDFが付属している場合があります（技術書など）。
  お持ちであれば 03_Bookall/ に保存してパスを教えてください。

【方法2】Kindle for PC から自動キャプチャする場合（推奨）
  ※ 個人利用の範囲に限ります
  Kindle for PC デスクトップアプリで書籍を開き、
  Claude Code が pyautogui で自動スクリーンショット取得 → PDF変換を実行します。

  自動キャプチャを実行しますか？（Y/N）

【方法3】手動でスクリーンショットを取得する場合
  Kindle for PC で書籍を開き、手動で各ページをキャプチャしてください。
```

#### ユーザーが自動化を希望した場合（Y）→ 方法2
Kindle for PC デスクトップアプリを pyautogui で自動操作する。
詳細な手順は `references/03-pdf-acquisition-guide.md` の「自動化ワークフロー」を参照。

**前提条件:**
- Kindle for PC がインストール済みであること
- 対象の書籍がダウンロード済みであること
- Python + pyautogui + Pillow がインストール済みであること

**自動化手順:**

1. ユーザーに確認する：
   ```
   以下の準備をお願いします：
   1. Kindle for PC を起動する
   2. 対象の書籍を開く
   3. 書籍の最初のページを表示する
   4. 準備ができたら「OK」と教えてください
   また、書籍のおおよそのページ数を教えてください。
   ```

2. 保存先フォルダを作成する：
   ```bash
   mkdir -p "${BOOKALL_DIR}/[書籍タイトル]"
   ```

3. pyautogui を使った自動キャプチャスクリプトを実行する：
   ```bash
   python -c "
   import pyautogui
   import pygetwindow as gw
   import time
   import os
   import hashlib

   save_dir = '${BOOKALL_DIR}/[書籍タイトル]'
   total_pages = [ユーザーが指定したページ数]
   wait_sec = 1.5
   refocus_interval = 10  # N ページごとに Kindle を再フォーカス
   duplicate_threshold = 3  # 連続重複がこの回数に達したら自動停止

   # Kindleウィンドウを自動検出・アクティブ化・最大化
   kindle_windows = gw.getWindowsWithTitle('Kindle')
   if not kindle_windows:
       print('ERROR: Kindleウィンドウが見つかりません。Kindle for PCを起動してください。')
       exit(1)
   kindle_win = kindle_windows[0]
   kindle_win.activate()
   time.sleep(0.5)
   kindle_win.maximize()
   time.sleep(1)

   prev_hash = None
   dup_count = 0

   for i in range(1, total_pages + 1):
       # 定期的に Kindle ウィンドウを再フォーカス
       if i % refocus_interval == 0:
           kindle_win.activate()
           time.sleep(0.3)

       # スクリーンショット取得
       screenshot = pyautogui.screenshot()

       # 重複検出（画像ハッシュ比較）
       img_hash = hashlib.md5(screenshot.tobytes()).hexdigest()
       if img_hash == prev_hash:
           dup_count += 1
           if dup_count >= duplicate_threshold:
               print(f'同一画像が {duplicate_threshold} 回連続 → 最終ページと判断し停止（{i - duplicate_threshold} ページ）')
               # 重複分のファイルを削除
               for d in range(duplicate_threshold):
                   dup_file = os.path.join(save_dir, f'page_{i - d:03d}.png')
                   if os.path.exists(dup_file):
                       os.remove(dup_file)
               break
       else:
           dup_count = 0
       prev_hash = img_hash

       filepath = os.path.join(save_dir, f'page_{i:03d}.png')
       screenshot.save(filepath)
       print(f'Captured page {i}/{total_pages}')

       # ページ送り（左矢印キー = 日本語書籍の次ページ）
       pyautogui.press('left')
       time.sleep(wait_sec)

   print('Capture completed.')
   "
   ```

   **自動化機能:**
   - Kindleウィンドウの自動検出・フォーカス・最大化（手動操作不要）
   - 10ページごとにKindleウィンドウを再フォーカス（フォーカス喪失対策）
   - 同一画像が3回連続で検出されたら最終ページと判断して自動停止（重複防止）

   **注意:**
   - 日本語書籍は右→左の読み順のため、`left` キーで次ページに進む
   - 横書き書籍の場合は `right` キーに変更する
   - キャプチャ中はマウス・キーボードに触れないこと

4. 全ページのキャプチャ完了後、Pillow で画像を PDF に結合する：
   ```bash
   python -c "
   from PIL import Image
   import glob
   imgs = sorted(glob.glob('${BOOKALL_DIR}/[書籍タイトル]/page_*.png'))
   images = [Image.open(f).convert('RGB') for f in imgs]
   images[0].save('${BOOKALL_DIR}/[書籍タイトル].pdf', save_all=True, append_images=images[1:])
   print('PDF created successfully.')
   "
   ```

5. 完成したPDFを `${BOOKALL_DIR}/[書籍タイトル].pdf` に保存する

6. → Step 3 へ（PDFモードで処理）

#### ユーザーが手動で用意する場合（N）→ 方法3
```
Kindle for PC で書籍を開き、各ページを手動でスクリーンショットしてください。
  - Windows: Win + Shift + S でSnipping Tool
  - ファイル名を page_001.png, page_002.png, ... の連番にする
  - 03_Bookall/[書籍タイトル]/ フォルダに保存する

準備ができたらフォルダパスを教えてください。
```
→ 準備完了後、Step 3 へ

---

## Step 3: 入力ファイルの読み込み

### PDF モードの場合
1. Read ツールで PDF を読み込む（大きなPDFは `pages` パラメータで分割読み込み）
2. 目次・章構成を把握する

### 画像モードの場合
1. Glob ツールで画像ファイル一覧を取得する
2. Read ツールで各画像を順番に読み込む
3. 画像内のテキストを認識し、章構成を把握する

---

## Step 4: 既存ハイライトの確認

`${HIGHLIGHT_DIR}` に書籍タイトルと一致するファイルがあるか確認する。

### ハイライトが存在する場合
- Read ツールでハイライトファイルを読み込む
- ハイライト箇所を要約に重点的に反映する（読者が重要と判断した箇所）

### ハイライトが存在しない場合
- PDF/画像の内容のみで要約を生成する

---

## Step 5: 章別要約の生成

書籍の各章について以下を生成する：

1. **章タイトル**
2. **要約**（500〜1000文字）
   - その章の核心的なメッセージ
   - 具体的な事例・データがあれば含める
   - ハイライトがある場合はその箇所を重点的に扱う

---

## Step 6: 全体サマリーの生成

章別要約をもとに以下を生成する：

1. **概要**（2〜3行）: 書籍全体を一言で表現
2. **主要な学び**（3〜5項目）: 書籍から得られる重要な知見
3. **アクションアイテム**（3〜5項目）: 読者が実践できる具体的なアクション

出力テンプレートの詳細は `references/01-summary-template.md` を参照。

---

## Step 7: Obsidianに保存

**CRITICAL: 必ず Write ツールでファイルを保存すること。チャット表示だけで終わらせてはいけない。**

1. **Bash ツールでディレクトリを確認する:**
   ```bash
   ls "${BOOKMEMO_DIR}"
   ```

2. **Write ツールで要約ファイルを保存する:**
   `${BOOKMEMO_DIR}/[書籍タイトル].md`

   ファイル内容は `references/01-summary-template.md` のテンプレートに従う。

3. **入力PDFを 03_Bookall にコピーする（PDFモードの場合）:**
   ```
   ユーザーに確認: 「元のPDFを 03_Bookall/ にコピーしますか？（Y/N）」
   ```
   承認された場合のみ Bash ツールで `cp` を実行する。

---

## Step 8: 完了通知

**CRITICAL: 生成した要約の全文をチャットに表示しない。完了通知のみ表示する。**

```
✅ 読書要約を生成しました

📖 書籍: [タイトル]
📁 保存先: 04_Bookmemo/[タイトル].md
📝 章数: [N]章
📎 ハイライト連携: [あり / なし]
🕐 生成日時: YYYY-MM-DD HH:MM

Obsidianで開いて確認してください。
```

---

## エラーハンドリング

### 入力ファイルが見つからない場合
```
❌ ファイルが見つかりません: [パス]
正しいファイルパスを確認して再度お知らせください。
```

### PDFが大きすぎて読み込めない場合
```
⚠️ PDFのページ数が多いため、分割して読み込みます。
処理に時間がかかる場合があります。
```
→ 20ページずつ分割して読み込み、順次処理する。

### 画像の文字認識が困難な場合
```
⚠️ 一部の画像でテキストの認識が困難です。
該当ページ: [ページ番号]
読み取れた範囲で要約を生成します。
```

### 同名の要約ファイルが既に存在する場合
```
⚠️ 04_Bookmemo/ に同名のファイルが既に存在します。
A. 上書きする
B. 別名で保存する（[タイトル]_v2.md）
どちらにしますか？
```
