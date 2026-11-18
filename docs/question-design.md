# Question Design / Aspect Ontology

## Design Principle

性能やジャンルを聞くのではない。

人が価値を感じる理由と、即冷める条件を聞く。

各 facet は原則 1 つの **meta-aspect** にぶら下がる。
対応が弱い facet は **domain-only** とし、横断説明に使わない。

- マッチ精度: domain facet
- 横断説明: primary meta のみ
- ベクトル本体: **能動取得**（アンケート / critique）。受動ログは検証用

---

# 取得モデル（重要）

## 混同しない二層

| 層 | 名前 | 聞く内容 | 保存先 |
|---|---|---|---|
| A | Item reason scores | 「このアイテムが好き／嫌いな理由の強さ」 | `review_answers` |
| B | Global importance | 「今後の選択でその軸をどれだけ重視するか」 | `user_aspect_weights` |
| C | Hard veto | 「これなら候補から外してよい」 | `user_hard_vetoes` |
| D | Experience goals | 「得たい体験」（少数） | `user_experience_goals` |

**禁止:** A の平均をそのまま B だと思わない。
好きな Ahri の機動力が高いことと、機動力を常に最重視することは別である。

導出ルール（初期）:

1. Seed で A（アイテム理由）を取る
2. 選ばれた facet について「これは今後も重視？」を聞き B を初期化（未回答は中立 5/10）
3. 嫌い理由のうち「絶対無理」を C に昇格（ユーザー確認必須）
4. D は最大 2 つまで選択式

## 能動 vs 受動

- **能動（ベクトルの正）:** Seed / active questions / critique / 構造化プロファイル編集
- **受動（検証・補助）:** 試合数、視聴完了、離脱、再訪 — 「当たったか」の計測。好き理由の代替にしない

---

# Shared Meta-Aspects

| Meta | 意味 | 体験の核 |
|---|---|---|
| Consistency | 内部整合 | 世界が嘘をついていないか |
| CraftDepth | 作り手の知識・密度 | 浅い／深い |
| Agency | 主導感 | 受け身か主導か |
| Clarity | ルール・因果の明瞭さ | 分かるか混乱か |
| Sensation | 快感・没入 | 気持ちよいか |
| Identity | 同一化 | 自分ごとになるか |
| Growth | 習熟・理解の深化 | 伸びている感覚 |

---

# Experience Goals（少数・選択式）

概念だけで終わらせない。Seed で最大 2 つ。

| Goal | 例（LoL） | 例（Movies） |
|---|---|---|
| mastery | 上手くなりたい | 考えるほど報われたい |
| flow_sensation | 操作が気持ちいい | 没入・感覚的快楽 |
| expression | プレイメイクで差を出したい | テーマ・美学の表明 |
| comfort | 安定して勝ちたい／疲れない | 安心して見られる |
| discovery | 新しい体験を試したい | 未知の世界に触れたい |

goals はランキングの軽い prior と Reason cards に使う。facet の代替ではない。

---

# Seed 予算（摩擦対策）

目標: **初回 5 分以内**。

| ステップ | 上限 |
|---|---|
| 好きなアイテム | 3 |
| 嫌いなアイテム | 2 |
| アイテムあたりの理由 facet | 最大 2 |
| hard veto | 最大 3 |
| experience goals | 最大 2 |
| グローバル重要度の追加確認 | 選ばれた facet のみ（最大 6） |

全 facet を埋めさせない。
不足分は active questions と推薦後 critique で埋める。

### Seed UI フロー

1. Primary context（LoL: メインロール 1 つ）
2. 好き 3 / 嫌い 2 を選択
3. 各アイテムで理由チップを最大 2（プリセット facet）
4. 「今後も重視？」で B を確認
5. 「絶対に無理」リストから veto を確認（C）
6. goals を最大 2

---

# Dual Question Types

## Soft（A/B）

1〜10。

- A: このアイテムについての理由の強さ
- B: グローバル重要度

## Hard veto（C）

boolean、または閾値。
ユーザー確認なしに A/B から自動昇格しない。

## Negative soft

嫌いだが即除外ではない。減点のみ。

---

# LoL Facets

LoL は **primary_role**（top / jungle / mid / adc / support）を必須コンテキストとする。
同一 facet でもロールが違う比較は confidence を下げる。

## Identity / Sensation

| Facet | Meta | 横断 | 質問例 |
|---|---|---|---|
| appearance | Identity | primary | 見た目が好き |
| skins | Sensation | weak / domain-leaning | スキンが好き |
| voice | Sensation | weak | ボイスが好き |
| character_lore | Identity | primary | キャラ設定が好き |
| world_fantasy | Identity | primary | 世界観が好き |

## Agency / Consistency

| Facet | Meta | 横断 | 質問例 |
|---|---|---|---|
| lane_strength | Agency | primary | レーン戦が強い |
| lane_priority | Agency | primary | 主導権が取れる |
| lane_stability | Consistency | primary | 安定している |
| teamfight_power | Agency | primary | 集団戦が強い |
| playmaking | Agency | primary | プレイメイクできる |
| pick | Agency | primary | ピック性能が高い |
| carry | Agency | primary | キャリー性能 |

## Sensation / Growth / Clarity

| Facet | Meta | 横断 | 質問例 |
|---|---|---|---|
| peel | Clarity | primary | ピールしやすい |
| kill_threat | Sensation | weak | キル性能 |
| snowball | Growth | weak | スノーボール性能 |
| combo_fun | Sensation | primary | コンボが楽しい |
| mobility | Sensation | primary | 機動力が高い |
| mechanical_difficulty | Growth | **domain-only** | 操作難易度（映画の Growth と同一視しない） |
| outplay | Agency | primary | 相手を翻弄できる |
| skill_expression | Growth | primary | 上手さが出る |
| mastery_growth | Growth | primary | 成長を感じられる |
| catharsis | Sensation | primary | 爽快感がある |

`domain-only` / `weak` は meta 説明カードに出さない。

---

# LoL Hard Veto / Negative Soft

## Hard veto

| Veto | Meta | 例 |
|---|---|---|
| ally_dependence | Agency | 味方依存が耐えられない |
| unbearable_complexity | Clarity | 操作が複雑すぎて楽しくない |
| instability_dealbreaker | Consistency | 不安定だと楽しめない |

適用条件: 対象 item の該当 facet に **confidence ≥ 0.6** があるときだけ除外。
未満は hard ではなく強い negative soft として減点。

## Negative soft

- 退屈
- 集団戦が苦手
- 難易度が高い（人による）

---

# Movies Facets

## Consistency（primary）

| Facet | 質問例 |
|---|---|
| lore_consistency | 世界設定の内部整合が高い |
| causal_coherence | 物語の因果が破綻していない |
| tonal_coherence | トーンが一貫している |

## CraftDepth（primary）

| Facet | 質問例 |
|---|---|
| lore_density | 設定・背景の密度が高い |
| directorial_competence | 監督・脚本の知識と手練れを感じる |
| craft_detail | ディテールの作り込みが深い |

## Agency / Clarity / Sensation / Identity / Growth

| Facet | Meta | 質問例 |
|---|---|---|
| protagonist_agency | Agency | 主人公が状況を動かせる |
| choice_salience | Agency | 選択や駆け引きが効いている |
| rule_clarity | Clarity | 作品内ルールが明瞭 |
| exposition_clarity | Clarity | 情報提示が分かりやすい |
| immersion | Sensation | 没入できる |
| sensory_pleasure | Sensation | 映像・音・テンポが気持ちよい |
| emotional_arc | Sensation | 感情曲線が効いている |
| character_attachment | Identity | キャラに同一化できる |
| world_attachment | Identity | 世界観に入り込みたい |
| intellectual_payoff | Growth | 考えるほど報われる |
| rewatch_discovery | Growth | 再視聴で理解が深まる |

---

# Movies Hard Veto

| Veto | Meta | 例 |
|---|---|---|
| lore_contradiction | Consistency | 世界観の致命的矛盾で即冷める |
| competence_collapse | CraftDepth | 知識不足が見えた時点で終了 |
| tonal_whiplash | Consistency | トーン破綻で没入が死ぬ |

Movies 開始時は LoL meta を **低信頼 prior** にするだけ。
映画用 Seed（好き/嫌い各少数 + veto）を必須とし、prior だけで推薦しない。

---

# Meta ↔ Domain Mapping Summary

| Meta | LoL（primary） | Movies（primary） |
|---|---|---|
| Consistency | lane_stability | lore_consistency, tonal_coherence |
| CraftDepth | （LoL は弱め。lore 密度は Identity 側） | lore_density, directorial_competence |
| Agency | lane_priority, playmaking, pick, carry | protagonist_agency, choice_salience |
| Clarity | peel | rule_clarity, exposition_clarity |
| Sensation | mobility, combo_fun, catharsis | immersion, sensory_pleasure |
| Identity | appearance, character_lore, world_fantasy | character_attachment, world_attachment |
| Growth | mastery_growth, skill_expression | intellectual_payoff, rewatch_discovery |

---

# Active Questions

未観測で、かつ候補分岐に効く facet / veto だけ聞く。

優先度:

1. Hard veto 未確定かつ除外が割れている軸
2. 上位候補のスコア差を生む未観測 facet
3. ロール文脈で意味が変わる facet

---

# Scrutable Profile

数値 + 構造化テキスト。

```text
Role: Mid
Goals: expression, mastery
Important: Agency (playmaking, pick), Consistency
Veto: ally_dependence
Less important: appearance
```

**NL 自由編集だけで再ランクしない。**
編集 UI は構造化フィールドを変更し、要約文はそれに追従する。
自由文は「提案された構造化差分」に変換 → ユーザー確認 → 適用。
