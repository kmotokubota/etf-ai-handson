# ETF AI ハンズオン
## Agentic AI for Portfolio Managers

Snowflake Cortex AI を使った **ETFポートフォリオ分析AIアシスタント** をゼロから構築するハンズオンコンテンツです。

---

## 概要

ETFのポートフォリオデータ・ファンドドキュメント・マーケットニュースを統合し、
自然言語でポートフォリオを分析できる AI アシスタント（Portfolio Copilot）を構築します。

**想定時間**: 約2時間  
**対象**: ポートフォリオマネージャー・運用チームメンバー  
**前提知識**: SQLの基礎、Snowflakeアカウントへのアクセス

---

## ハンズオン構成

| # | ファイル | 概要 | 所要時間 |
|---|---|---|---|
| 事前準備 | `setup.sql` | DB・スキーマ・テーブル・データ作成 | 10分 |
| Part 1 | `part1_data_setup.ipynb` | データ探索 & AI Functions（要約・感情分析・分類・抽出） | 35分 |
| Part 2 | `part2_cortex_analyst.ipynb` | Semantic View 作成 & Cortex Analyst（自然言語 to SQL） | 35分 |
| Part 3 | `part3_cortex_search.ipynb` | Cortex Search Service 構築 + Agent設定 | 25分 |
| デモ | Snowflake Intelligence | Portfolio Copilot でシナリオを体験 | 15分 |

---

## セットアップ

### 前提条件
- Snowflake アカウント（`ACCOUNTADMIN` ロール推奨）
- Snowsight へのアクセス

### 手順

#### Step 1 — setup.sql の実行

1. Snowsight の **SQL Editor** を開く
2. `setup.sql` の内容を貼り付けてすべて実行する
3. `ETF_AI_HANDSON_DB` / `DEMO_SCHEMA` と各テーブル・ウェアハウスが作成される

#### Step 2 — Workspace に Git リポジトリを追加

1. Snowsight の左メニューから **Workspace** を開く
2. 「**+ 追加**」→「**Git リポジトリから**」をクリック
3. 以下を入力する

   | 項目 | 値 |
   |---|---|
   | URL | `https://github.com/kmotokubota/etf-ai-handson` |
   | 認証 | **パブリックリポジトリ**として作成 |

4. 「作成」をクリックするとノートブックが Workspace に追加される

#### Step 3 — ノートブックを順番に実行

```
setup.sql（SQL Editor）
  ↓
part1_data_setup → part2_cortex_analyst → part3_cortex_search
  ↓
[手動操作] Cortex Agent 作成
  ↓
Snowflake Intelligence デモ
```

> **注意**: 各 Part は前の Part の実行結果に依存しています。必ず順番通りに実行してください。

### ウェアハウス

| ウェアハウス | サイズ | 用途 |
|---|---|---|
| `DEMO_WH` | XSMALL | SQL・AI Functions・Cortex Analyst |
| `COMPUTE_WH` | XSMALL | Cortex Search インデックス構築 |

---

## アーキテクチャ

```
ETFデータ
  ├── DIM_ETF（ETF銘柄マスタ）         10銘柄
  ├── DIM_PORTFOLIO（ポートフォリオ）    5本
  ├── FACT_HOLDINGS（保有明細）         21件
  ├── FACT_PERFORMANCE（月次実績）      48件
  ├── MARKET_NEWS（マーケットニュース）  12件
  └── ETF_DESCRIPTIONS（ファンド説明）  10件
        │
        ├── AI Functions ─────────── ニュース要約・センチメント・抽出（Part 1）
        │
        ├── Cortex Analyst ────────── 自然言語 to SQL（Part 2）
        │       └── PORTFOLIO_ANALYTICS_VIEW（Semantic View）
        │
        ├── Cortex Search ─────────── ETFドキュメント＆ニュース検索（Part 3）
        │       └── ETF_KNOWLEDGE_SEARCH
        │
        └── Cortex Agent ──────────── Snowflake Intelligence（手動設定）
                └── ETF_PORTFOLIO_COPILOT
```

---

## デモシナリオ（Portfolio Copilot）

### シナリオ 1: 朝のポートフォリオブリーフィング
```
グローバル分散成長ポートフォリオ（P004）の現在の保有状況と含み損益を教えてください。
また、最近のマーケットニュースでこのポートフォリオに影響しそうな情報はありますか？
```

### シナリオ 2: リスク分析
```
2025年の年間リターンが目標リターンに最も届いていないポートフォリオはどれですか？
そのポートフォリオの主要保有ETFについて、最近のネガティブなニュースを調べてください。
```

### シナリオ 3: ETF選定サポート
```
インカム重視でポートフォリオを構築したいと考えています。
ETF運用会社のインカム系ETFの一覧と、それぞれの特徴・配当利回りを比較してください。
```

### シナリオ 4: 関税ショックへの対応
```
米国の関税強化ニュースがありましたが、
現在保有しているポートフォリオの中で最も影響を受けそうなETFはどれですか？
具体的な比重と含み損益も合わせて教えてください。
```

---

## Snowflake オブジェクト一覧

| 種別 | オブジェクト名 | 役割 |
|---|---|---|
| Database | `ETF_AI_HANDSON_DB` | ハンズオン用データベース |
| Schema | `DEMO_SCHEMA` | データテーブルスキーマ |
| Schema | `AI` | AIコンポーネントスキーマ |
| Warehouse | `DEMO_WH` | 通常処理用ウェアハウス |
| Warehouse | `COMPUTE_WH` | Cortex Search 構築用 |
| Table | `DIM_ETF` | ETF銘柄マスタ（ETF運用会社 10本） |
| Table | `DIM_PORTFOLIO` | ポートフォリオマスタ（5本） |
| Table | `FACT_HOLDINGS` | 保有ETF明細 |
| Table | `FACT_PERFORMANCE` | 月次パフォーマンス実績 |
| Table | `MARKET_NEWS` | マーケットニュース（12件） |
| Table | `ETF_DESCRIPTIONS` | ETFファンド説明文書 |
| Semantic View | `PORTFOLIO_ANALYTICS_VIEW` | ポートフォリオ自然言語クエリ |
| Cortex Search | `ETF_KNOWLEDGE_SEARCH` | ETFドキュメント＆ニュース検索 |
| Cortex Agent | `ETF_PORTFOLIO_COPILOT` | 統合AIアシスタント（手動作成） |

---

## 体験ポイント

### AI Functions（Part 1）
> 「2,000文字のニュース記事が2行に。毎朝の情報収集が10分から30秒に。」

### Cortex Analyst（Part 2）
> 「SQLを書かずに、日本語で『含み損が最も大きいポートフォリオは？』と聞くだけで答えが返ってくる。」

### Cortex Search（Part 3）
> 「『リスクを抑えながら収益を得たい』という言葉でインカム系ETFのドキュメントがヒットする。」

### Portfolio Copilot（デモ）
> 「ポートフォリオの数値分析とETFドキュメント検索を1つのUIで。
> Snowflakeにデータを集めるだけで、AIが自動的に最適なツールを選択して回答する。」

---

## 関連リポジトリ

- [sfguide-agentic-ai-for-asset-management-ja](https://github.com/kmotokubota/sfguide-agentic-ai-for-asset-management-ja) - SAM デモ（フルバージョン）
- [cortex-ai-handson](https://github.com/kmotokubota/cortex-ai-handson) - 証券営業インテリジェンスハンズオン
