# 03 — PDF入手・スクリーンショット自動化ガイド

## PDF入手方法の分岐

```
Kindle書籍のPDFが必要
  │
  ├─ PDF付属の書籍 → そのまま 03_Bookall/ に保存
  │  （技術書・O'Reilly等で購入時にPDFが付属するケース）
  │
  └─ PDF非付属の書籍 → スクリーンショットからPDF化
     │
     ├─ 自動化（推奨: Kindle for PC + pyautogui）
     └─ 手動（ユーザーが自分でキャプチャ）
```

---

## 方法1: PDF付属の書籍

Kindle購入時にPDFファイルが付属している場合（主に技術書）：

1. 付属PDFをダウンロードする
2. `03_Bookall/[書籍タイトル].pdf` に保存する
3. → 要約処理へ進む

---

## 方法2: Kindle for PC + pyautogui 自動キャプチャ（推奨）

一般的なKindle書籍ではPDFが提供されていないため、
**Kindle for PC（デスクトップアプリ）** を使ってスクリーンショットを自動取得し、PDFに変換する。

**CRITICAL: この作業は個人利用の範囲に限る。`references/02-copyright-notice.md` を参照。**

### 前提条件

- **Kindle for PC** がインストール済み（Microsoft Store または Amazon 公式サイトからダウンロード）
- 対象の書籍がダウンロード済み（オフライン閲覧可能な状態）
- **Python 3** がインストール済み
- **pyautogui** と **Pillow** がインストール済み

### 使用するアプリ

| アプリ | 入手先 | 説明 |
|--------|--------|------|
| **Kindle for PC** | Microsoft Store / Amazon公式 | Windowsデスクトップ版Kindleリーダー。ダウンロード済み書籍をオフラインで閲覧可能 |

### 必要なPythonライブラリ

```bash
pip install pyautogui Pillow
```

### 自動化ワークフロー

#### ステップ1: ユーザー準備

ユーザーに以下を依頼する：
1. Kindle for PC を起動する
2. 対象の書籍を開く
3. 書籍の最初のページを表示する
4. 準備ができたら「OK」と教えてもらう
5. 書籍のおおよそのページ数を教えてもらう

※ ウィンドウのフォーカス・最大化はスクリプトが自動で行う

#### ステップ2: 保存先フォルダの作成

```bash
mkdir -p "${BOOKALL_DIR}/[書籍タイトル]"
```

#### ステップ3: 自動キャプチャスクリプトの実行

pyautogui を使って全ページを自動キャプチャする：

```python
import pyautogui
import pygetwindow as gw
import time
import os
import hashlib

save_dir = '${BOOKALL_DIR}/[書籍タイトル]'
total_pages = [ユーザーが指定したページ数]
wait_sec = 1.5  # ページ送り後の待機秒数
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

    # スクリーンショット取得（画面全体）
    screenshot = pyautogui.screenshot()

    # 重複検出（画像ハッシュ比較）
    img_hash = hashlib.md5(screenshot.tobytes()).hexdigest()
    if img_hash == prev_hash:
        dup_count += 1
        if dup_count >= duplicate_threshold:
            actual = i - duplicate_threshold
            print(f'同一画像が {duplicate_threshold} 回連続 → 最終ページと判断し停止（{actual} ページ）')
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

    # ページ送り
    pyautogui.press('left')  # 日本語書籍（縦書き）: left = 次ページ
    time.sleep(wait_sec)

print('Capture completed.')
```

**ページ送りキーの選択:**

| 書籍の種類 | 読み順 | 次ページキー |
|-----------|--------|-------------|
| 日本語書籍（縦書き） | 右→左 | `left` |
| 横書き書籍（技術書等） | 左→右 | `right` |

**自動化機能:**
- Kindleウィンドウの自動検出・フォーカス・最大化（手動操作不要）
- 10ページごとにKindleウィンドウを再フォーカス（フォーカス喪失対策）
- 同一画像が3回連続で検出されたら最終ページと判断して自動停止（重複防止）

**注意事項:**
- キャプチャ中はマウス・キーボードに触れないこと（pyautogui が操作を奪われる）
- wait_sec はデフォルト1.5秒。画像が多い書籍やPCスペックが低い場合は2〜3秒に延長

#### ステップ4: PDF変換

全ページのキャプチャ完了後、Pillow で画像を PDF に結合する：

```python
from PIL import Image
import glob

# 画像ファイルを連番順にソート
imgs = sorted(glob.glob('${BOOKALL_DIR}/[書籍タイトル]/page_*.png'))

# PDF に結合
images = [Image.open(f).convert('RGB') for f in imgs]
images[0].save(
    '${BOOKALL_DIR}/[書籍タイトル].pdf',
    save_all=True,
    append_images=images[1:]
)
print('PDF created successfully.')
```

#### ステップ5: 完了

```
✅ PDF作成完了
📁 保存先: 03_Bookall/[書籍タイトル].pdf
📄 ページ数: [N]ページ
```

→ SKILL.md の Step 3 へ進む（PDFモードで処理）

---

## 方法3: 手動キャプチャ

自動化を希望しない場合の手順：

1. **Kindle for PC** で書籍を開く
2. **各ページのスクリーンショットを取得する**
   - Windows: `Win + Shift + S` でSnipping Tool
   - フルスクリーン: `PrintScreen` キー
3. **ファイル名を連番に統一する**
   - 例: `page_001.png`, `page_002.png`, ...
4. **画像を `03_Bookall/[書籍タイトル]/` フォルダに保存する**
5. **画像フォルダのパスをスキルに渡す**
   - → 画像モードで要約処理が可能

---

## 保存先の規則

| 種類 | 保存先 |
|------|--------|
| 付属PDF | `03_Bookall/[書籍タイトル].pdf` |
| スクリーンショット画像 | `03_Bookall/[書籍タイトル]/page_NNN.png` |
| 変換済みPDF | `03_Bookall/[書籍タイトル].pdf` |

---

## トラブルシューティング

### pyautogui がインストールされていない
```bash
pip install pyautogui
```
または
```bash
python -m pip install pyautogui
```

### Pillow がインストールされていない
```bash
pip install Pillow
```

### キャプチャ画像にKindle以外のウィンドウが映り込む
- Kindle for PC を最大化してからスクリプトを実行する
- 他のウィンドウを最小化しておく
- マルチモニター環境の場合、pyautogui はプライマリモニターをキャプチャする

### ページ送りが反応しない
- Kindle for PC のウィンドウにフォーカスがあることを確認する
- スクリプト開始前に `time.sleep(5)` を追加し、手動でKindleウィンドウをクリックする時間を確保する
- `left` / `right` キーの代わりに `pagedown` / `pageup` を試す

### キャプチャが途中で止まる
- ページ数を多めに設定しておき、余分な画像は後で削除する
- wait_sec を2〜3秒に延長する

### ページ数が不明
- Kindle for PC の画面下部にあるページ進捗バーから推定する
- 書籍のAmazon商品ページで「ページ数」を確認する
- わからない場合は多めに設定し、最終ページに到達後に停止する
