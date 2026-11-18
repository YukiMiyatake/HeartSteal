# HeartSteal Concept

## Mission

人自身も言語化しきれていない「本当に好きな体験」と「即冷める条件」を発見する。

---

## Definition

HeartSteal は推薦サービスではない。

ドメイン横断の **Preference Operating System** である。

「なぜ好きか／なぜ即冷めるか」を

- 表現し
- 照合し
- 更新する

基盤として設計する。

LoL と Movies は同じエンジン上の **ドメインパック** である。

---

## Existing Problem

### Behavior-first recommenders

LoL ツールは勝率・ピック率・使用チャンピオンから薦める。
Netflix 型はジャンルと視聴ログの平均から薦める。

どちらも「何に触れたか」は分かっても、

- なぜ好きだったか
- なぜ途中で冷めたか

を扱わない。

同じ Ahri メインでも、同じ SF ファンでも、効いている軸は全く異なる。

### Soft attributes and dealbreakers

人はしばしばジャンルではなく、測りにくい指数で判断する。

例（映画）:

- 世界設定の致命的矛盾
- 監督・脚本の知識不足
- トーンの破綻

例（LoL）:

- 味方依存が耐えられない
- 操作負荷が楽しさを殺す
- プレイメイクできないと退屈

これらは平均視聴完了や勝率では捉えにくい。

### Chat-only failure

素のチャット AI は主観語の個人意味（「面白い」「重い」「整合している」）を取り違えやすい。
自由対話だけでは dealbreaker の特定も非効率である。

---

## HeartSteal Approach

分析対象は「好きなアイテム」ではない。

1. **Hard veto** … 違反したら候補から外す
2. **Soft aspects** … 相対的に強い／弱い好み
3. **Experience goals** … 得たい体験

好みと拒否を分離し、説明可能に照合する。

---

## Dual Preference Model

ユーザーは単一の平均値ベクトルを持たない。

- 層 A `item reason` … このアイテムが好き／嫌いな理由
- 層 B `soft_weights` … 今後の選択でのグローバル重要度（A の平均ではない）
- 層 C `hard_vetoes` … dealbreaker（ユーザー確認済みのみ）
- 層 D `experience_goals` … 得たい体験（最大 2）
- `negative_soft` … 嫌いだが即除外ではない軸
- 構造化プロファイル + NL 要約 … **正本は構造化。NL 自由編集は確認後に反映**

映画の「設定矛盾で即冷め」は soft の低点ではなく hard veto として扱う。

---

## Acquisition Policy

嗜好ベクトルの正は **能動取得** である。

- Seed / active questions / critique / 構造化編集 → ベクトル生成・更新
- 試合数・視聴完了・離脱 → 成功検証（好き理由の代替にしない）

Netflix 型の視聴時間平均では、測りにくい指数も dealbreaker も取れない。

詳細な問題点と改善は [design-notes.md](design-notes.md)。

---

## Guardrails（設計で潰した落とし穴）

- 未観測 facet を 0 埋めして類似ユーザーを作らない
- item スコアの confidence が低い veto は除外せず減点
- meta の弱い対応で映画を説明しない
- LoL→Movies は低信頼 prior + 映画 Seed 必須
- フィードバックで veto を勝手に外さない

---

## Cross-Domain Sharing

共有してよいもの:

- meta-aspects（Consistency, Agency, CraftDepth など）
- ユーザーの自己モデル（soft / hard / nl_profile）
- 推薦パイプラインとフィードバック学習

共有してはいけないもの:

- チャンピオン名や映画タイトルの安易な直訳
- facet の機械的な同一視（例: LoL の pick ≈ 映画の twist）

横断の売りは「アイテムの共通化」ではなく **自己モデルの持ち歩き** である。

よい説明:

```text
あなたは Consistency と Agency を極端に重視する。
この作品は Consistency が低いので除外した。
```

悪い説明:

```text
Hwei が好きだからこの SF が好き。
```

---

## Example (LoL)

Ahri が好き

↓

主導権: 10 / ピック: 10 / 見た目: 2
味方依存: hard veto

↓

Hard filter で味方依存の強い候補を除外

↓

Aspect match + 類似ユーザー

↓

Hwei / Lissandra / Taliyah を提案

↓

「Agency と pick を重視し、依存を拒否するユーザーの 72% が…」と説明する

---

## Example (Movies)

ある SF が好き

↓

設定密度: 高 / 知識の厚み: 高 / 感情曲線: 中
世界観矛盾: hard veto

↓

矛盾の多い候補を除外し、CraftDepth / Consistency で照合

↓

理由付きで候補を返す

---

## Core Philosophy

拒否は切る。
好みは分解する。
説明は読めて直せる。
ドメインはパックで差し替える。

HeartSteal は価値観を理解する Preference OS である。
