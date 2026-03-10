# URL定義・パラメータ一覧

## 1. 日本株 — かぶたん（kabutan.jp）

### ベースURL・ランキング種別

| ランキング | ベースURL | modeパラメータ |
|---|---|---|
| 値上がり率 | `https://kabutan.jp/warning/?mode=2_1` | `mode=2_1` |
| 値下がり率 | `https://kabutan.jp/warning/?mode=2_2` | `mode=2_2` |
| 出来高 | `https://kabutan.jp/warning/volume_ranking` | — |
| 売買代金 | `https://kabutan.jp/warning/trading_value_ranking` | — |

### 市場パラメータ

| 市場 | パラメータ | 備考 |
|---|---|---|
| 東証全体 | `market=0` または パラメータなし | ETF/REIT含む可能性あり → 後処理フィルタ必要 |
| プライム | `market=1` | ETF自動除外 |
| スタンダード | `market=2` | ETF自動除外 |
| グロース | `market=3` | ETF自動除外 |

### ページネーション

- `page=1`（1〜50件）、`page=2`（51〜100件）
- 表示件数指定不可（最大50件/ページ）

### URL組み立てルール

**値上がり率・値下がり率:**
```
https://kabutan.jp/warning/?mode={mode}&market={market}&page={page}
```

例:
```
# プライム 値上がり率 1ページ目
https://kabutan.jp/warning/?mode=2_1&market=1&page=1

# プライム 値上がり率 2ページ目
https://kabutan.jp/warning/?mode=2_1&market=1&page=2

# スタンダード 値下がり率 1ページ目
https://kabutan.jp/warning/?mode=2_2&market=2&page=1
```

**出来高・売買代金:**
```
https://kabutan.jp/warning/volume_ranking?market={market}&page={page}
https://kabutan.jp/warning/trading_value_ranking?market={market}&page={page}
```

例:
```
# プライム 出来高 1ページ目
https://kabutan.jp/warning/volume_ranking?market=1&page=1

# グロース 売買代金 2ページ目
https://kabutan.jp/warning/trading_value_ranking?market=3&page=2
```

### 全URL一覧（日本株: 32URL）

```
# ── プライム（market=1）──
kabutan.jp/warning/?mode=2_1&market=1&page=1     # 値上がり率 p1
kabutan.jp/warning/?mode=2_1&market=1&page=2     # 値上がり率 p2
kabutan.jp/warning/?mode=2_2&market=1&page=1     # 値下がり率 p1
kabutan.jp/warning/?mode=2_2&market=1&page=2     # 値下がり率 p2
kabutan.jp/warning/volume_ranking?market=1&page=1           # 出来高 p1
kabutan.jp/warning/volume_ranking?market=1&page=2           # 出来高 p2
kabutan.jp/warning/trading_value_ranking?market=1&page=1    # 売買代金 p1
kabutan.jp/warning/trading_value_ranking?market=1&page=2    # 売買代金 p2

# ── スタンダード（market=2）──
kabutan.jp/warning/?mode=2_1&market=2&page=1
kabutan.jp/warning/?mode=2_1&market=2&page=2
kabutan.jp/warning/?mode=2_2&market=2&page=1
kabutan.jp/warning/?mode=2_2&market=2&page=2
kabutan.jp/warning/volume_ranking?market=2&page=1
kabutan.jp/warning/volume_ranking?market=2&page=2
kabutan.jp/warning/trading_value_ranking?market=2&page=1
kabutan.jp/warning/trading_value_ranking?market=2&page=2

# ── グロース（market=3）──
kabutan.jp/warning/?mode=2_1&market=3&page=1
kabutan.jp/warning/?mode=2_1&market=3&page=2
kabutan.jp/warning/?mode=2_2&market=3&page=1
kabutan.jp/warning/?mode=2_2&market=3&page=2
kabutan.jp/warning/volume_ranking?market=3&page=1
kabutan.jp/warning/volume_ranking?market=3&page=2
kabutan.jp/warning/trading_value_ranking?market=3&page=1
kabutan.jp/warning/trading_value_ranking?market=3&page=2

# ── 東証全体（market指定なし or market=0）──
kabutan.jp/warning/?mode=2_1&page=1
kabutan.jp/warning/?mode=2_1&page=2
kabutan.jp/warning/?mode=2_2&page=1
kabutan.jp/warning/?mode=2_2&page=2
kabutan.jp/warning/volume_ranking?page=1
kabutan.jp/warning/volume_ranking?page=2
kabutan.jp/warning/trading_value_ranking?page=1
kabutan.jp/warning/trading_value_ranking?page=2
```

### ETFフィルタ（東証全体の場合）

東証全体（market未指定）で取得した場合、以下の市場コードを持つ銘柄を除外する:
- `東Ｅ` — 東証ETF
- `東EN` — 東証ETN
- `東Ｒ` — 東証REIT
- `東IF` — 東証インフラファンド

保持する市場コード:
- `東Ｐ` — 東証プライム
- `東Ｓ` — 東証スタンダード
- `東Ｇ` — 東証グロース

### 取得データ項目（かぶたん）

| 項目 | 説明 | 例 |
|---|---|---|
| コード | 銘柄コード（4桁） | 7203 |
| 銘柄名 | 会社名 | トヨタ自動車 |
| 市場 | 市場区分 | 東Ｐ |
| 株価 | 現在値（円） | 2,850 |
| 前日比 | 前日比（円） | +120 |
| 変動率 | 前日比（%） | +4.40% |
| 出来高 | 出来高（株） | 15,230,000 |
| PER | 株価収益率（倍） | 12.5 |
| PBR | 株価純資産倍率（倍） | 1.2 |
| 利回り | 配当利回り（%） | 2.8 |

---

## 2. 米国株 — Yahoo!ファイナンス（finance.yahoo.co.jp）

### ベースURL・ランキング種別

| ランキング | URL |
|---|---|
| 値上がり率 | `https://finance.yahoo.co.jp/stocks/us/ranking/up` |
| 値下がり率 | `https://finance.yahoo.co.jp/stocks/us/ranking/down` |
| 出来高 | `https://finance.yahoo.co.jp/stocks/us/ranking/volume` |
| 売買代金 | `https://finance.yahoo.co.jp/stocks/us/ranking/tradingValue` |

### 市場パラメータ

| 市場 | パラメータ |
|---|---|
| 全市場 | `market=all` |
| NYSE | `market=NYSE` |
| NASDAQ | `market=NASDAQ` |

### ページネーション

- `page=1`（1〜50件）、`page=2`（51〜100件）
- 1ページあたり50件表示

### URL組み立てルール

```
https://finance.yahoo.co.jp/stocks/us/ranking/{type}?market={market}&page={page}
```

例:
```
# 全市場 値上がり率 1ページ目
https://finance.yahoo.co.jp/stocks/us/ranking/up?market=all&page=1

# NYSE 出来高 2ページ目
https://finance.yahoo.co.jp/stocks/us/ranking/volume?market=NYSE&page=2

# NASDAQ 売買代金 1ページ目
https://finance.yahoo.co.jp/stocks/us/ranking/tradingValue?market=NASDAQ&page=1
```

### 全URL一覧（米国株: 24URL）

```
# ── 全市場（market=all）──
finance.yahoo.co.jp/stocks/us/ranking/up?market=all&page=1
finance.yahoo.co.jp/stocks/us/ranking/up?market=all&page=2
finance.yahoo.co.jp/stocks/us/ranking/down?market=all&page=1
finance.yahoo.co.jp/stocks/us/ranking/down?market=all&page=2
finance.yahoo.co.jp/stocks/us/ranking/volume?market=all&page=1
finance.yahoo.co.jp/stocks/us/ranking/volume?market=all&page=2
finance.yahoo.co.jp/stocks/us/ranking/tradingValue?market=all&page=1
finance.yahoo.co.jp/stocks/us/ranking/tradingValue?market=all&page=2

# ── NYSE ──
finance.yahoo.co.jp/stocks/us/ranking/up?market=NYSE&page=1
finance.yahoo.co.jp/stocks/us/ranking/up?market=NYSE&page=2
finance.yahoo.co.jp/stocks/us/ranking/down?market=NYSE&page=1
finance.yahoo.co.jp/stocks/us/ranking/down?market=NYSE&page=2
finance.yahoo.co.jp/stocks/us/ranking/volume?market=NYSE&page=1
finance.yahoo.co.jp/stocks/us/ranking/volume?market=NYSE&page=2
finance.yahoo.co.jp/stocks/us/ranking/tradingValue?market=NYSE&page=1
finance.yahoo.co.jp/stocks/us/ranking/tradingValue?market=NYSE&page=2

# ── NASDAQ ──
finance.yahoo.co.jp/stocks/us/ranking/up?market=NASDAQ&page=1
finance.yahoo.co.jp/stocks/us/ranking/up?market=NASDAQ&page=2
finance.yahoo.co.jp/stocks/us/ranking/down?market=NASDAQ&page=1
finance.yahoo.co.jp/stocks/us/ranking/down?market=NASDAQ&page=2
finance.yahoo.co.jp/stocks/us/ranking/volume?market=NASDAQ&page=1
finance.yahoo.co.jp/stocks/us/ranking/volume?market=NASDAQ&page=2
finance.yahoo.co.jp/stocks/us/ranking/tradingValue?market=NASDAQ&page=1
finance.yahoo.co.jp/stocks/us/ranking/tradingValue?market=NASDAQ&page=2
```

### ETFフィルタ（米国株）

Yahoo!ファイナンスにはETF除外フィルタがないため、取得後に以下の方法で除外する:

1. **銘柄名による除外**: 銘柄名に以下のキーワードが含まれるものを除外
   - `ETF`
   - `ETN`
   - `Trust`（ただし「Real Estate Investment Trust」等の一般企業は誤除外に注意）
   - `Fund`
   - `iShares`
   - `Vanguard`
   - `SPDR`
   - `ProShares`
   - `Invesco`

2. **ティッカーパターンによる除外**: ETF/ETNに多いティッカーパターン
   - 一般的にETFは3〜4文字のティッカーが多いが、通常株も同様のため銘柄名での判定を優先

3. **除外精度の注意**: 100%の除外精度は保証されない。除外した銘柄数をログに記録する。

### 取得データ項目（Yahoo!ファイナンス）

| 項目 | 説明 | 例 |
|---|---|---|
| 順位 | ランキング順位 | 1 |
| 銘柄名 | 会社名 | Apple Inc. |
| ティッカー | ティッカーシンボル | AAPL |
| 市場 | 取引所 | NASDAQ |
| 取引値 | 現在値（ドル） | 185.50 |
| 前日比 | 前日比（ドル） | +3.20 |
| 変動率 | 前日比（%） | +1.76% |
| 出来高 | 出来高（株） | 52,340,000 |
