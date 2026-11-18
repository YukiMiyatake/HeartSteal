# Recommendation Engine

## Goal

もっとも強い／もっとも再生されやすいアイテムを探すのではない。

**もっとも継続しやすい体験**を探す。

拒否条件を守り、好みを分解し、理由を説明し、外れから学習する。

---

## Acquisition Policy

| 信号 | 役割 |
|---|---|
| Seed / active questions / critique / 構造化編集 | **嗜好ベクトルの正（能動）** |
| 興味・継続の自己申告 | 学習ラベル |
| 試合数・視聴完了・離脱（受動） | 成功検証・体験量。好き理由の代替ではない |

Netflix 型の視聴時間平均で soft_weights を作らない。

---

## Input

### Positive Data

- Item reason scores（層 A）
- Global importance（層 B）
- Experience goals（層 D）

### Negative Data

negative soft と嫌い理由

### Hard Veto Data

ユーザー確認済み dealbreaker（層 C）

### Feedback Data

興味・体験量・継続・critique・veto 的中

### Catalog Data

`item_aspect_scores`（confidence / version 付き）

---

## Dual User Model

```json
{
  "domain": "lol",
  "primary_role": "mid",
  "experience_goals": ["expression", "mastery"],
  "soft_weights": {
    "playmaking": 0.95,
    "pick": 0.92,
    "mobility": 0.80,
    "appearance": 0.20
  },
  "observed_mask": {
    "playmaking": true,
    "pick": true,
    "mobility": true,
    "appearance": true,
    "teamfight_power": false
  },
  "meta_weights": {
    "Agency": 0.93,
    "Consistency": 0.78,
    "Sensation": 0.55,
    "Identity": 0.22
  },
  "meta_confidence": {
    "Agency": 0.8,
    "Consistency": 0.5
  },
  "hard_vetoes": [
    { "facet": "ally_dependence", "op": ">=", "threshold": 7 }
  ],
  "negative_soft": {
    "boring": 0.7
  },
  "nl_profile": "Agency と pick を重視。味方依存は即拒否。"
}
```

### ベクトル生成手順

1. Seed の層 A から「触れた facet」を列挙
2. 層 B（グローバル重要度）で `soft_weights` を設定。未確認 facet は **未観測**（0 埋めしない）
3. primary meta のみ `meta_weights` を集約。weak / domain-only は含めない
4. `observed_mask` を保存
5. hard veto は埋め込みに混ぜず別テーブル
6. ドメイン別 embedding は **観測済み soft_weights の正規化ベクトル**

---

## Item Model

```text
ItemAspectVector {
  domain, item_id,
  facets: { name: score, confidence },
  meta: { ... primary only ... },
  catalog_version,
  sources: [curated | community | llm_extract | riot]
}
```

### Bootstrap（必須）

Phase 1 で LoL 主要チャンピオンに **curated facet** を入れる。
community 平均は confidence を低くし、件数に応じて上げる。

`confidence` 減衰:

- 新しい catalog_version / パッチで未再評価なら confidence を下げる
- 閾値未満の facet は hard filter に使わない

---

## Hybrid Pipeline（5段）

### Step 1 — Hard filter（confidence-aware）

veto 条件に該当し、かつ item 側 `confidence >= 0.6` なら除外。

confidence 不足時は除外せず、強いペナルティ（negative soft 相当）に落とす。

誤除外を、出し損ないより重く見る。

### Step 2 — Aspect match

観測済み soft_weights のみで加重類似。

```text
score = sum_i w_i * item_i * mask_i / sum_i w_i * mask_i
      - negative_soft_penalty
      + goal_prior
```

- 主計算は domain facet
- meta は説明と低信頼 prior
- LoL は primary_role が近い item を軽くブースト（ロール必須ではないが比較信頼度が上がる）

### Step 3 — Similar users（masked similarity）

同ドメイン近傍。cosine は **共通観測次元のみ**。

```text
similarity = cosine(u_obs, v_obs on shared mask)
require |shared_mask| >= 3
```

条件未満なら collaborative boost = 0（Aspect match のみで返す）。

### Step 4 — Novelty / coverage

未体験優先、同一クラスタ過集中抑制、探索枠少数。

### Step 5 — Reason cards

primary meta / 観測済み facet / veto 非抵触だけを説明に使う。
weak 対応や未観測次元は出さない。

```text
効いている軸: playmaking, pick
Goals: expression
Veto: ally_dependence 非抵触（confidence 0.8）
```

---

## Pipeline Diagram

```text
candidates
   ↓
Hard filter (veto ∧ confidence)
   ↓
Aspect match (masked weights)
   ↓
Similar users boost (shared mask >= 3)
   ↓
Novelty / coverage
   ↓
Reason cards (primary only)
   ↓
recommendations
```

---

## Conflict Resolution（自己申告 vs フィードバック）

| 種類 | 規則 |
|---|---|
| Hard veto | ユーザー明示編集以外で自動解除しない。フィードバックは「解除提案」まで |
| Soft weights | フィードバックで低学習率更新（例: 0.1〜0.2）。大きく動かすときは確認 |
| Item reason（A） | 履歴として残す。B を上書きしない |
| 受動ログ | 成功／失敗ラベル。B の直接代入に使わない |
| NL 自由文 | 構造化差分に変換 → 確認後に適用。直接再ランクしない |

---

## Cross-Domain Prior

LoL → Movies:

1. primary meta_weights を **low confidence prior** としてコピー
2. Movies Seed を必須実施
3. 映画 facet の観測が N 件（初期目安 4）を超えた meta から prior を上書き
4. Reason cards で「ゲームからの仮の傾向」と明示し、確定扱いにしない

---

## Elicitation & Critique Loop

### Seed

5 分予算。好き 3 / 嫌い 2 / 理由各最大 2 / veto 最大 3 / goals 最大 2 / role 1

### Active questions

候補分岐に効く未観測軸のみ

### Critique

構造化チップ優先（「もっと Agency」「難易度が高すぎ」）。
自由文は構造化候補へ変換して確認。

### Online learning

```text
推薦 → 興味 → 体験量 → 継続 → critique/veto
 → soft を低学習率更新 / veto 解除は提案のみ
```

---

## Success / Failure

### Success

```text
興味あり + 十分な体験量 + 継続したい
```

- LoL: 10 試合以上
- Movies: 視聴完了または再訪

### Failure

- 興味あり + 未体験
- 体験 + 継続したくない
- veto すべきものを推薦した（特に高 confidence なのに見逃し）

---

## Ranking Blend（初期）

```text
final =
  0.50 * aspect_match
+ 0.25 * similar_user_score   # 条件未達なら 0、aspect 側へ再配分
+ 0.15 * novelty
+ 0.10 * goal_prior
```

Cold start（類似ユーザー不可）時:

```text
final = 0.75 * aspect_match + 0.15 * novelty + 0.10 * goal_prior
```

---

## Future

- Contextual Bandit
- Soft-attribute 個人意味（CAV）
- Critique 文構造化（LLM はここまで）
