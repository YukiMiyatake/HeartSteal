# Roadmap

Preference OS として、LoL でコアを固め、Movies Pack で横断を検証する。

設計上の穴（二層 elicitation、curated catalog、confidence veto、mask 類似など）は Core から必須。

---

## Phase 1 — Core v0（LoL）

目標

動く Preference Core。能動取得でベクトルを作り、信頼度付きハイブリッド推薦まで。

必須

- Google Login
- domains / items
- **主要チャンピオンの curated facet カタログ**（confidence 高め）
- Seed（5分予算）: 好き3/嫌い2、理由各≤2、veto≤3、goals≤2、primary role
- 二層モデル: item reason（A）≠ global importance（B）
- hard veto（user_confirmed のみ）
- observed_mask 付き Embedding
- Aspect match（本体）+ Similar users（shared mask≥3 のときのみ）
- confidence-aware Hard filter
- Reason cards（primary meta/facet のみ）
- **構造化プロファイル編集**（再ランクに直結）

リリース目標

Closed Beta

非目標（この Phase ではやらない）

- NL 自由文の直接再ランク
- Movies 自動移植推薦
- Riot による嗜好推定

---

## Phase 2 — Feedback v1

目標

自己申告と結果の衝突を安全に学習する。

機能

- 興味 / 使用 / 継続
- critique（構造化チップ）
- veto_triggered / veto_miss
- soft の低学習率更新
- veto 解除・追加は **提案→確認**
- catalog_version に応じた item confidence 減衰

成功定義

```text
興味あり + 10試合以上 + 継続したい
```

---

## Phase 3 — Movies Pack v0

目標

同一コアに映画を載せ、横断 prior を過信せず検証する。

機能

- Movies facets / vetoes
- 小カタログ curated scores
- ドメイン切替 UI
- LoL meta を **low confidence prior** としてのみ利用
- **映画用 Seed 必須**（prior だけで薦めない）
- 説明に「ゲームからの仮の傾向」ラベル

検証指標

初期納得度。精度より先に「説明が自分に合うか」。

---

## Phase 4 — Scrutable NL

目標

自然言語要約を便利にする。ランクの正本は構造化のまま。

機能

- structured → NL 要約生成
- 自由文 → 構造化差分の提案
- 確認後にのみ weights/vetoes 更新
- 編集履歴（user_profile_snapshots）

---

## Phase 5 — Soft-attribute Learning

機能

- タグ / レビュー / critique からの soft attribute
- 個人意味の差
- active questioning の本格化（情報利得）

---

## Phase 6 — Bandit / Online + Riot

機能

- Contextual Bandit
- Riot API で体験量の自動追跡（嗜好の代替ではない）
- Premium 候補

---

## Phase 7 — Analytics & Additional Packs

- 嗜好変化 / シーズン比較 / クラスタ
- Anime / Music / Books

---

## Final Vision

拒否は切る（ただし自信があるときだけ硬く）。
好みは分解する（アイテム理由と重要度を混ぜない）。
説明は読めて直せる（構造化が正、NL は窓）。
ドメインはパックで差し替える（meta prior は低信頼）。
