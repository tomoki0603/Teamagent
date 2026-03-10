# Excelテンプレート・シート構成定義

## 1. ファイル構成

### 出力ファイル

| ファイル | ファイル名 | シート数 |
|---|---|---|
| 日本株 | `JP_株式ランキング_YYYY-MM-DD.xlsx` | 16シート |
| 米国株 | `US_株式ランキング_YYYY-MM-DD.xlsx` | 12シート |

### 保存先
```
C:\Users\start\Desktop\
```

---

## 2. 日本株 Excel シート構成（16シート）

### シート命名規則
```
{市場名}_{ランキング種別}
```

### シート一覧

| No | シート名 | 市場 | ランキング |
|---|---|---|---|
| 1 | 東証全体_値上がり率 | 東証全体 | 値上がり率 |
| 2 | 東証全体_値下がり率 | 東証全体 | 値下がり率 |
| 3 | 東証全体_出来高 | 東証全体 | 出来高 |
| 4 | 東証全体_売買代金 | 東証全体 | 売買代金 |
| 5 | プライム_値上がり率 | プライム | 値上がり率 |
| 6 | プライム_値下がり率 | プライム | 値下がり率 |
| 7 | プライム_出来高 | プライム | 出来高 |
| 8 | プライム_売買代金 | プライム | 売買代金 |
| 9 | スタンダード_値上がり率 | スタンダード | 値上がり率 |
| 10 | スタンダード_値下がり率 | スタンダード | 値下がり率 |
| 11 | スタンダード_出来高 | スタンダード | 出来高 |
| 12 | スタンダード_売買代金 | スタンダード | 売買代金 |
| 13 | グロース_値上がり率 | グロース | 値上がり率 |
| 14 | グロース_値下がり率 | グロース | 値下がり率 |
| 15 | グロース_出来高 | グロース | 出来高 |
| 16 | グロース_売買代金 | グロース | 売買代金 |

### 日本株 カラム定義

| 列 | ヘッダー | データ型 | 幅 | 説明 |
|---|---|---|---|---|
| A | 順位 | 整数 | 8 | ランキング順位（1〜100） |
| B | コード | 文字列 | 10 | 銘柄コード（4桁） |
| C | 銘柄名 | 文字列 | 25 | 会社名 |
| D | 市場 | 文字列 | 8 | 市場区分（東Ｐ/東Ｓ/東Ｇ） |
| E | 株価 | 数値 | 12 | 現在値（円） |
| F | 前日比 | 数値 | 12 | 前日比（円） |
| G | 変動率(%) | 数値(%) | 12 | 前日比率（%） |
| H | 出来高 | 数値 | 15 | 出来高（株） |
| I | PER | 数値 | 10 | 株価収益率（倍） |
| J | PBR | 数値 | 10 | 株価純資産倍率（倍） |
| K | 利回り(%) | 数値(%) | 10 | 配当利回り（%） |

---

## 3. 米国株 Excel シート構成（12シート）

### シート一覧

| No | シート名 | 市場 | ランキング |
|---|---|---|---|
| 1 | 全市場_値上がり率 | 全市場 | 値上がり率 |
| 2 | 全市場_値下がり率 | 全市場 | 値下がり率 |
| 3 | 全市場_出来高 | 全市場 | 出来高 |
| 4 | 全市場_売買代金 | 全市場 | 売買代金 |
| 5 | NYSE_値上がり率 | NYSE | 値上がり率 |
| 6 | NYSE_値下がり率 | NYSE | 値下がり率 |
| 7 | NYSE_出来高 | NYSE | 出来高 |
| 8 | NYSE_売買代金 | NYSE | 売買代金 |
| 9 | NASDAQ_値上がり率 | NASDAQ | 値上がり率 |
| 10 | NASDAQ_値下がり率 | NASDAQ | 値下がり率 |
| 11 | NASDAQ_出来高 | NASDAQ | 出来高 |
| 12 | NASDAQ_売買代金 | NASDAQ | 売買代金 |

### 米国株 カラム定義

| 列 | ヘッダー | データ型 | 幅 | 説明 |
|---|---|---|---|---|
| A | 順位 | 整数 | 8 | ランキング順位（1〜100） |
| B | ティッカー | 文字列 | 10 | ティッカーシンボル |
| C | 銘柄名 | 文字列 | 30 | 会社名 |
| D | 市場 | 文字列 | 10 | 取引所（NYSE/NASDAQ） |
| E | 取引値($) | 数値 | 12 | 現在値（ドル） |
| F | 前日比($) | 数値 | 12 | 前日比（ドル） |
| G | 変動率(%) | 数値(%) | 12 | 前日比率（%） |
| H | 出来高 | 数値 | 15 | 出来高（株） |

---

## 4. スタイル定義

### ヘッダー行（1行目）

```python
header_font = Font(name='Yu Gothic', size=11, bold=True, color='FFFFFF')
header_fill = PatternFill(start_color='2B5797', end_color='2B5797', fill_type='solid')
header_alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
header_border = Border(
    left=Side(style='thin'),
    right=Side(style='thin'),
    top=Side(style='thin'),
    bottom=Side(style='medium')
)
```

### データ行（2行目以降）

```python
data_font = Font(name='Yu Gothic', size=10)
data_alignment = Alignment(vertical='center')
data_border = Border(
    left=Side(style='thin'),
    right=Side(style='thin'),
    top=Side(style='thin'),
    bottom=Side(style='thin')
)

# 偶数行の背景色（縞模様）
even_row_fill = PatternFill(start_color='F2F7FC', end_color='F2F7FC', fill_type='solid')
```

### 条件付き書式

```python
# 変動率がプラスの場合: 赤文字
positive_font = Font(color='CC0000')

# 変動率がマイナスの場合: 青文字
negative_font = Font(color='0066CC')
```

### 数値フォーマット

```python
# 株価・取引値
price_format = '#,##0'

# 変動率
rate_format = '0.00%'

# 出来高
volume_format = '#,##0'

# PER/PBR
ratio_format = '0.0'

# 利回り
yield_format = '0.00%'
```

---

## 5. シート共通設定

```python
# ヘッダー行を固定（スクロール時に常に表示）
ws.freeze_panes = 'A2'

# オートフィルター設定
ws.auto_filter.ref = ws.dimensions

# 印刷設定
ws.page_setup.orientation = 'landscape'
ws.page_setup.fitToPage = True
ws.page_setup.fitToWidth = 1
ws.page_setup.fitToHeight = 0
```

---

## 6. ②将来拡張を考慮したデータ設計

### 分析用の統一インデックス

各シートの **B列（コード/ティッカー）** を主キーとして、
異なる日付のファイル間で銘柄を紐づけ可能にする。

### pandas での読み込み例

```python
import pandas as pd
from pathlib import Path

# 特定日の日本株プライム値上がり率を読み込み
df = pd.read_excel(
    'JP_株式ランキング_2026-03-09.xlsx',
    sheet_name='プライム_値上がり率'
)

# 複数日のデータを時系列で結合
dfs = []
for f in sorted(Path('.').glob('JP_株式ランキング_*.xlsx')):
    date = f.stem.split('_')[-1]  # YYYY-MM-DD
    df = pd.read_excel(f, sheet_name='プライム_値上がり率')
    df['日付'] = date
    dfs.append(df)

timeline = pd.concat(dfs, ignore_index=True)

# 特定銘柄のランキング推移
stock_history = timeline[timeline['コード'] == '7203']
```
