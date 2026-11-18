# HeartSteal

> あなたが好きなのはアイテムではない。
> あなたが求めている体験だ。
> そして、あなたが即冷める理由もだ。

HeartSteal はドメイン横断の **Preference Operating System** です。

推薦カタログを巨大化するサービスではありません。
「なぜ好きか／なぜ即冷めるか」を表現し、照合し、更新する基盤です。

最初のドメインパックは League of Legends。
同じコアの上に Movies（および将来の Anime / Music / Books）を載せます。

---

# 一句の定義

HeartSteal is not a recommendation service.

HeartSteal is a Preference OS that discovers why people love — and instantly reject — what they experience.

---

# 既存ツールの限界

## LoL ツール

勝率・ピック率・レーンなど客観データからおすすめする。

同じ Ahri メインでも

- 機動力が好き
- ピック性能が好き
- プレイメイクが好き
- キャラ設定が好き

理由は全く異なる。

## Netflix 型

ジャンルや視聴ログの平均嗜好に最適化する。

しかし人によっては

- 世界設定の矛盾
- 作り手の知識不足
- トーンの破綻

といった **測りにくい指数** で即冷める。
ジャンル一致だけではマッチしない。

## 素のチャット AI

自由対話だけでは、主観語の個人意味や dealbreaker を効率よく特定できない。

---

# Preference OS の核

HeartSteal が変える一点:

「好きな理由のベクトルで似た人を探す」

から

**「拒否は切る・好みは分解する・説明は読めて直せる・ドメインはパックで差し替える」**

へ。

| 層 | 役割 | 映画の例 | LoL の例 |
|---|---|---|---|
| Hard veto | 違反したら候補除外 | 世界観の致命的矛盾 | 味方依存が耐えられない |
| Soft aspects | 相対的な好みの強さ | 設定密度、知識の厚み | 主導権、ピック、機動力 |
| Experience goals | 得たい体験 | 知的充足、没入 | プレイメイク、成長実感 |

横断で共有するのはチャンピオン名やジャンルではない。
**Consistency / Agency / CraftDepth** などの meta-aspect と、ユーザー自身のプロファイルである。

---

# Dual Preference Model

ユーザーは単一ベクトルではなく次を持つ。

- 層 A … このアイテムの好き／嫌い理由
- 層 B `soft_weights` … 今後のグローバル重要度（A の平均ではない）
- 層 C `hard_vetoes` … dealbreaker（確認済みのみ）
- 層 D `experience_goals` … 得たい体験（最大 2）
- 構造化プロファイル … 再ランクの正本（NL は窓）

映画の「設定矛盾で即冷め」は soft の低点ではなく **hard veto**。

## ベクトルの取り方

**能動が正、受動は検証。**

- アンケート Seed（5分）/ 追加質問 / critique / 構造化編集 → ベクトル
- 試合数・視聴完了 → 当たったかの計測（嗜好の代替にしない）

---

# 推薦パイプライン（要約）

1. Hard filter（veto ∧ item confidence）
2. Aspect match（観測済み次元のみ）
3. Similar users（共通観測が十分なときだけ）
4. Novelty / coverage
5. Reason cards（primary 軸のみ）

協調フィルタ単独には依存しない。
ユーザー密度が薄い初期でも aspect match で動く。

詳細は [docs/recommendation-engine.md](docs/recommendation-engine.md)。

---

# MVP（LoL Core v0）

## ユーザー登録

Googleログイン

## 短時間 Elicitation（≤5分）

- primary role
- 好き 3 / 嫌い 2
- 各最大 2 の「なぜ」
- 今後も重視するか（グローバル重要度）
- hard veto 最大 3（確認必須）
- experience goals 最大 2

## カタログ

主要チャンピオンの curated facet（confidence 付き）が必須。

## 深層プロファイル

A/B/C/D を分離。未観測は埋めない。

## ハイブリッド推薦

Aspect match 本体 + 条件付き類似ユーザー + 理由表示。

## フィードバック

興味・使用量・継続・critique。
veto 変更は提案→確認。

---

# 技術スタック

## Frontend

- Next.js
- TypeScript
- Tailwind CSS

## Hosting

- Vercel

## Database

- Neon PostgreSQL

## Vector Search

- pgvector

## ORM

- Drizzle ORM

初期は LLM を推薦本体に使わない。
プロファイル要約と批判文の構造化に限定する。

---

# ドメイン展開

| モード | 役割 |
|---|---|
| HeartSteal LoL | 楔。Closed Beta でコア検証 |
| HeartSteal Movies | 2つ目のドメインパック。自己モデルの持ち歩きを検証 |
| Anime / Music / Books | 同一コア上の追加パック |

横断で約束すること:

- 「あなたは整合性と主導感を極端に重視する。この作品は整合性が低いので除外」

約束しないこと:

- 「LoL の Hwei が好きだからこの SF が好き」（facet の勝手な直訳）

---

# ドキュメント

- [docs/concept.md](docs/concept.md) … Preference OS の思想
- [docs/design-notes.md](docs/design-notes.md) … 設計上の問題点と改善
- [docs/question-design.md](docs/question-design.md) … 取得モデルと facet 対応
- [docs/recommendation-engine.md](docs/recommendation-engine.md) … 5段パイプライン
- [docs/architecture.md](docs/architecture.md) … 一般化スキーマ
- [docs/roadmap.md](docs/roadmap.md) … Core → Movies Pack → Soft-attribute learning

---

# Vision

拒否は切る。
好みは分解する。
説明は読めて直せる。
ドメインはパックで差し替える。
