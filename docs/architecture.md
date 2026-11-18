# Architecture

## Tech Stack

### Frontend

- Next.js
- TypeScript
- Tailwind CSS

### Backend

- Next.js Route Handlers
- Server Actions

### Hosting

- Vercel

### Database

- Neon PostgreSQL

### Vector Search

- pgvector

### ORM

- Drizzle ORM

初期は LLM を推薦本体に使わない。
プロファイル要約と critique 文の構造化提案に限定する。

---

## High Level Architecture

```text
User
 ↓
Next.js
 ↓
Preference Core API
 ↓
PostgreSQL + pgvector

Preference Core
├ Domains / Items
├ Aspects (meta + facet, transfer_strength)
├ User soft weights / hard vetoes / goals
├ Item aspect scores (confidence, version)
├ Domain embeddings (masked)
├ Recommendations + feedback
└ Structured profiles (+ NL summary)

Domain Packs
├ LoL Pack (role context, curated seed catalog)
└ Movies Pack
```

---

## Database Tables

### users

```sql
id
name
created_at
```

### domains

```sql
id
slug          -- lol | movies
name
created_at
```

### items

```sql
id
domain_id
external_key
name
role          -- LoL primary role hint 等
image_url
metadata jsonb
created_at
```

### aspects

```sql
id
domain_id     -- NULL = shared meta
parent_id
kind          -- meta | facet | goal | veto_facet
slug
name
description
transfer_strength -- primary | weak | domain_only
created_at
```

### questions

```sql
id
aspect_id
domain_id
prompt
question_type -- item_reason | global_importance | hard_veto | negative_soft | goal
scale_min
scale_max
created_at
```

### item_reviews

```sql
id
user_id
item_id
stance        -- like | dislike | neutral
created_at
```

### review_answers

```sql
id
review_id
question_id
score         -- 層A: このアイテムについての理由
created_at
```

層 A。グローバル重要度はここに入れない。

### user_aspect_weights

```sql
user_id
aspect_id
domain_id
weight        -- 層B: グローバル重要度
observed      -- false なら未観測（0埋め禁止）
updated_at
```

### user_experience_goals

```sql
user_id
domain_id
goal_aspect_id
rank          -- 1..2
created_at
```

### user_hard_vetoes

```sql
id
user_id
aspect_id
domain_id
op
threshold
active
source        -- user_confirmed | suggested
created_at
```

`suggested` はフィードバックからの解除／追加提案。適用は user_confirmed のみ。

### user_contexts

```sql
user_id
domain_id
key           -- primary_role 等
value
updated_at
```

### item_aspect_scores

```sql
item_id
aspect_id
score
confidence
source        -- curated | community | llm_extract | riot
catalog_version
updated_at
```

Hard filter は `confidence >= 0.6` のときのみ適用可能。

### catalog_versions

```sql
id
domain_id
label         -- patch 14.x / movies_catalog_2026_03
created_at
```

新 version 投入後、未再評価スコアは confidence 減衰ジョブの対象。

### user_embeddings

```sql
user_id
domain_id
embedding vector(256)
embedding_kind -- facet_observed | meta_summary
observed_dim_count
updated_at
```

類似検索は共通観測次元を前提にする（アプリ側で mask 処理、または次元を固定スロット＋別途 mask jsonb）。

### user_profile_snapshots

```sql
id
user_id
domain_id
structured_json  -- weights, vetoes, goals, role
nl_body
source           -- generated | user_confirmed_edit
version
created_at
```

自由文編集は直接 score に繋がない。
構造化差分確認後に `user_confirmed_edit` として保存し、weights を更新する。

### recommendations

```sql
id
user_id
item_id
domain_id
score
aspect_match_score
similar_user_score
novelty_score
reason_json jsonb
created_at
```

### recommendation_feedback

```sql
id
recommendation_id
interest_score
usage_level
retention_score
feedback_type
critique_aspects jsonb
veto_triggered boolean
veto_miss boolean
created_at
```

---

## Embedding Strategy

1. 観測済み `user_aspect_weights` のみベクトル化
2. meta_summary は primary かつ観測由来のみ
3. hard veto は埋め込みに混ぜない
4. 未観測次元は 0 埋めしない（mask 外）

Similar users:

```sql
SELECT *
FROM user_embeddings
WHERE domain_id = :domain_id
  AND embedding_kind = 'facet_observed'
  AND observed_dim_count >= 3
ORDER BY embedding <=> :target_embedding
LIMIT 50;
```

アプリ側で shared mask < 3 の相手は捨てる。

---

## Matching Flow

1. veto ∧ item.confidence >= 0.6 で除外（未満は減点）
2. masked aspect match + goal prior + role prior
3. shared-mask similar users（不可なら skip）
4. novelty
5. reason_json（primary のみ）
6. feedback は soft を低学習率更新、veto は提案のみ
7. 構造化プロファイル確認編集で即再計算

---

## Domain Pack Boundary

Domain Pack:

- items / curated seed catalog
- domain facets・questions
- role 等の context 定義
- 体験量定義
- catalog_version 運用

Preference Core:

- meta-aspects（transfer_strength 付き）
- dual model + goals
- confidence-aware hard filter
- masked hybrid matcher
- conflict resolution
- structured profile / NL summary
