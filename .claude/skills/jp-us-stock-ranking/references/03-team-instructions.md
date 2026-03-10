# チーム構成・作業指示

## 1. チーム概要

| 役割 | 担当 | サブエージェントタイプ |
|---|---|---|
| リーダー | 全体進行管理・Excel生成・保存 | —（自身） |
| JP担当 | かぶたんから日本株ランキング取得 | general-purpose |
| US担当 | Yahoo!ファイナンスから米国株ランキング取得 | general-purpose |

### チーム作成コマンド

```
TeamCreate:
  team_name: stock-ranking
  description: 日本株・米国株ランキングデータ取得チーム
```

---

## 2. リーダーの作業手順

### Step 1: チーム作成・メンバー起動

1. `TeamCreate` でチーム `stock-ranking` を作成
2. JP担当とUS担当を `Task` ツールで **同時に** 起動（並列実行）

```
Task (JP担当):
  name: jp-stock-fetcher
  subagent_type: general-purpose
  team_name: stock-ranking
  prompt: [JP担当への指示（後述）]

Task (US担当):
  name: us-stock-fetcher
  subagent_type: general-purpose
  team_name: stock-ranking
  prompt: [US担当への指示（後述）]
```

**CRITICAL**: 2つの Task を同一メッセージ内で呼び出し、並列実行すること。

### Step 2: 結果受信

各担当からの結果メッセージを受信する。
結果はJSON形式またはテキスト形式で、各ランキングのデータを含む。

### Step 3: Excel生成

openpyxl を使用したPythonスクリプトを生成・実行する。
- `references/02-excel-templates.md` のシート構成・スタイルに従う
- JP/US別ファイルとして出力

### Step 4: 保存・クリーンアップ

1. デスクトップにExcelファイルを保存
2. 各チームメイトに `SendMessage` (type: shutdown_request) を送信
3. `TeamDelete` でチームを削除
4. ユーザーに完了報告

---

## 3. JP担当への指示テンプレート

以下をJP担当のTask promptとして使用する:

```
あなたは日本株ランキングデータの取得担当です。

## タスク
かぶたん（kabutan.jp）から以下の16パターンのランキングデータを取得してください。

## 対象
- 市場: 東証全体、プライム(market=1)、スタンダード(market=2)、グロース(market=3)
- ランキング: 値上がり率(mode=2_1)、値下がり率(mode=2_2)、出来高(volume_ranking)、売買代金(trading_value_ranking)
- 各100銘柄（50件×2ページ）

## URL構造

値上がり率・値下がり率:
https://kabutan.jp/warning/?mode={mode}&market={market}&page={page}

出来高:
https://kabutan.jp/warning/volume_ranking?market={market}&page={page}

売買代金:
https://kabutan.jp/warning/trading_value_ranking?market={market}&page={page}

東証全体はmarketパラメータなし:
https://kabutan.jp/warning/?mode={mode}&page={page}
https://kabutan.jp/warning/volume_ranking?page={page}
https://kabutan.jp/warning/trading_value_ranking?page={page}

## 取得データ項目
各銘柄から以下を抽出:
- コード（4桁）、銘柄名、市場区分、株価、前日比、変動率(%)、出来高、PER、PBR、利回り

## ETFフィルタ
- プライム/スタンダード/グロース: ETFは自動除外されるのでフィルタ不要
- 東証全体: 市場コードが「東Ｅ」「東EN」「東Ｒ」「東IF」の銘柄を除外し、
  「東Ｐ」「東Ｓ」「東Ｇ」のみ保持する

## 出力形式
取得したデータを以下の形式でリーダーに送信:
- 市場名とランキング種別をキーとした構造化データ
- 各エントリに順位(1〜100)を付与

## 注意事項
- WebFetch で取得すること
- 1ページ50件 × 2ページで100件取得
- 取得できない場合はエラー内容を報告
- データが100件に満たない場合は取得できた件数で報告
```

---

## 4. US担当への指示テンプレート

以下をUS担当のTask promptとして使用する:

```
あなたは米国株ランキングデータの取得担当です。

## タスク
Yahoo!ファイナンス（finance.yahoo.co.jp）から以下の12パターンのランキングデータを取得してください。

## 対象
- 市場: 全市場(market=all)、NYSE(market=NYSE)、NASDAQ(market=NASDAQ)
- ランキング: 値上がり率(up)、値下がり率(down)、出来高(volume)、売買代金(tradingValue)
- 各100銘柄（50件×2ページ）

## URL構造
https://finance.yahoo.co.jp/stocks/us/ranking/{type}?market={market}&page={page}

typeの値:
- 値上がり率: up
- 値下がり率: down
- 出来高: volume
- 売買代金: tradingValue

例:
https://finance.yahoo.co.jp/stocks/us/ranking/up?market=all&page=1
https://finance.yahoo.co.jp/stocks/us/ranking/volume?market=NYSE&page=2

## 取得データ項目
各銘柄から以下を抽出:
- ティッカー、銘柄名、市場（NYSE/NASDAQ）、取引値($)、前日比($)、変動率(%)、出来高

## ETFフィルタ
Yahoo!ファイナンスにはETF除外フィルタがないため、取得後に以下の銘柄を除外:
- 銘柄名に以下のキーワードが含まれるもの:
  ETF, ETN, Fund, iShares, Vanguard, SPDR, ProShares, Invesco
- 除外した銘柄数を記録

## 出力形式
取得したデータを以下の形式でリーダーに送信:
- 市場名とランキング種別をキーとした構造化データ
- 各エントリに順位(1〜100)を付与
- ETF除外で100件を下回った場合はその旨を報告

## 注意事項
- WebFetch で取得すること
- 1ページ50件 × 2ページで100件取得
- 取得できない場合はエラー内容を報告
```

---

## 5. エラー時の対応フロー

### データ取得失敗

```
JP/US担当 → リーダーにエラー報告
リーダー → 失敗したパターンのみリトライ指示
         → 3回失敗で該当パターンをスキップ
         → ユーザーに部分的な結果を報告
```

### チームメイトが応答しない場合

```
リーダー → 60秒待機後にリトライメッセージ送信
        → 応答なしの場合は該当担当の作業を自身で実施
```

### ファイル保存エラー

```
リーダー → PermissionError の場合:
          「Excelファイルが開かれています。閉じてから『続けて』と入力してください。」
        → その他のエラー: エラー内容をユーザーに報告
```

---

## 6. 並列実行の効率化ポイント

### WebFetch の並列呼び出し

各担当エージェント内でも、複数のWebFetchを同一メッセージで呼び出すことで並列化できる。

```
# 1回のメッセージで複数URL同時取得（推奨）
WebFetch: kabutan.jp/warning/?mode=2_1&market=1&page=1
WebFetch: kabutan.jp/warning/?mode=2_1&market=1&page=2
WebFetch: kabutan.jp/warning/?mode=2_2&market=1&page=1
WebFetch: kabutan.jp/warning/?mode=2_2&market=1&page=2
```

### 推奨バッチ分割

一度に大量のWebFetchを呼ぶとエラーになる可能性があるため、
市場ごとにバッチ分割を推奨:

```
バッチ1: プライム全ランキング（8 URL）
バッチ2: スタンダード全ランキング（8 URL）
バッチ3: グロース全ランキング（8 URL）
バッチ4: 東証全体全ランキング（8 URL）
```
