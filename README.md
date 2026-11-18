# HeartSteal

> あなたが好きなのはチャンピオンではない。
> あなたが求めている体験だ。

HeartSteal は League of Legends プレイヤー向けの深層嗜好分析プラットフォームです。

既存のチャンピオン推薦サービスは、

- 使用チャンピオン
- 勝率
- ピック率
- レーン

などの客観的データからおすすめを提示します。

HeartStealは異なります。

私たちは

「なぜそのチャンピオンが好きなのか」

を分析し、プレイヤー自身も気付いていない本質的な価値観を発見することを目的とします。

---

# 問題

例えば Ahri を好むプレイヤーがいた場合、

既存サービスでは

```text
Ahri
↓
LeBlanc
```

のような推薦が行われます。

しかし本当に重要なのは、

```text
なぜ Ahri が好きなのか
```

です。

プレイヤーによって理由は全く異なります。

例:

- 機動力が好き
- ピック性能が好き
- プレイメイクが好き
- 見た目が好き
- キャラクター設定が好き

同じ Ahri メインでも価値観は異なります。

HeartSteal はその違いを分析します。

---

# コンセプト

HeartStealが分析するのは

「好きなチャンピオン」

ではなく

「好きだと感じる理由」

です。

---

# MVP

## ユーザー登録

Googleログイン

---

## チャンピオン評価

ユーザーは好きなチャンピオンや嫌いなチャンピオンを登録します。

各チャンピオンに対し、

以下のような項目を評価します。

### 例

- 機動力
- 主導権
- ピック性能
- プレイメイク
- 集団戦
- レーン戦
- キル性能
- キャリー力
- 安定性
- 爽快感
- 難易度
- 見た目
- キャラクター設定

---

## 深層プロファイル生成

ユーザーごとに価値観ベクトルを生成します。

例:

```json
{
  "playmaking": 0.91,
  "pick": 0.95,
  "mobility": 0.84,
  "burst": 0.75
}
```

---

## 類似プレイヤー検索

同じチャンピオンを使っているかではなく、

同じ価値観を持つプレイヤーを発見します。

例:

```text
あなたと93%一致
```

---

## チャンピオン推薦

類似プレイヤーが高評価しているチャンピオンを推薦します。

例:

```text
1. Hwei
2. Lissandra
3. Taliyah
```

---

# 推薦後フィードバック

HeartStealが重視するのは推薦の結果です。

推薦後に以下を収集します。

### 興味度

1〜5

---

### 使用状況

- 未使用
- 1〜2試合
- 3〜10試合
- 10〜30試合
- 30試合以上

---

### 継続利用意向

1〜5

---

### 実際どうだったか

- 予想より良かった
- 予想通り
- 予想より悪かった

---

# システム思想

HeartStealは

「何が好きか」

より

「なぜ好きなのか」

を重要視します。

さらに

「なぜ推薦が外れたのか」

を学習し続けます。

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

---

# 将来構想

League of Legendsで構築した推薦基盤を利用し、

- HeartSteal Movies
- HeartSteal Music
- HeartSteal Anime
- HeartSteal Books

へ展開する。

最終的にはジャンルを超えた深層嗜好分析プラットフォームを目指す。

---

# Vision

HeartSteal is not a recommendation service.

HeartSteal is a system that discovers why people love what they love.
