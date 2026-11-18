# Design Notes — 問題点と改善

Preference OS 初版ドキュメントに対する設計レビュー結果。
以下は **採用した改善** であり、各本体ドキュメントへ反映済み。

---

## 見つかった問題

| # | 問題 | 影響 |
|---|---|---|
| 1 | soft が「重視度」なのか「そのアイテムの理由スコア」なのか曖昧 | ベクトルの意味が崩壊する |
| 2 | item facet スコアの供給源が弱い（鶏卵） | Aspect match が初期から動かない |
| 3 | meta 対応が無理矢理（例: 難易度→Growth） | 横断説明が嘘になる |
| 4 | アンケート摩擦の予算が未定義 | 離脱でプロファイルが空になる |
| 5 | 自己申告とフィードバック衝突時の優先順位がない | 学習が発散／veto が勝手に外れる |
| 6 | LoL のロール文脈がない | 同じ「機動力」が ADC/Support で別物 |
| 7 | 主観 item スコアに hard veto を無条件適用 | 誤除外が致命傷 |
| 8 | Movies への meta prior 移植が過信 | 横断の売りが炎上する |
| 9 | NL 編集→再ランクの変換規則がない | scrutable が飾りになる |
| 10 | 疎な facet ベクトルで cosine | 類似ユーザーが不安定 |
| 11 | Experience goals が概念だけで非モデル | 3層のつもりが実質2層 |
| 12 | パッチ／時期で item スコアが腐る | LoL で特に深刻 |

---

## 採用した改善（要約）

1. **二層 elicitation** … アイテム理由スコア ≠ グローバル重要度。別フィールド。
2. **能動が正、受動は検証** … ベクトル本体はアンケート／critique。試合数・視聴完了は成功計測。
3. **Seed 予算** … 5分以内、facet 選択はアイテムあたり最大2、veto 最大3。
4. **Curated seed catalog** … Phase 1 で主要チャンピオンに人手 facet を必須化。
5. **Veto は confidence 付き** … item スコア信頼度が閾値未満なら除外せず減点に落とす。
6. **meta は primary のみ横断** … 弱い対応は domain-only。説明に使わない。
7. **ロール条件** … LoL は primary role を取り、facet 比較をロール整合で補正。
8. **衝突解決** … veto はユーザー明示編集以外で自動解除しない。soft は低学習率でフィードバック更新。
9. **疎ベクトル** … 観測マスク上の重み付き類似。未観測次元は0埋めしない。
10. **横断 prior** … meta は低信頼 prior。Movies 開始時に必ず映画用 seed を取る。
11. **NL 編集** … テキスト直接では再ランクしない。構造化差分の確認後に適用。
12. **item スコア版** … patch/catalog version を持ち、古くなったら confidence を減衰。

詳細は question-design / recommendation-engine / architecture / roadmap を正本とする。
