# Recommendation Engine

## Goal

もっとも勝てるチャンピオンを探すのではない。

もっとも継続しやすいチャンピオンを探す。

---

## Input

### Positive Data

ユーザー評価

### Negative Data

嫌いな理由

### Feedback Data

推薦後評価

---

## User Vector

例

{
  mobility: 0.8,
  playmaking: 0.95,
  lane_priority: 0.9,
  difficulty: 0.6
}

---

## Step 1

ユーザーEmbedding生成

---

## Step 2

類似ユーザー抽出

Top 50

---

## Step 3

未プレイチャンピオン取得

---

## Step 4

類似ユーザー評価でランキング

---

## Step 5

推薦

---

# Feedback Loop

重要

---

推薦

↓

興味度

↓

使用

↓

継続利用

↓

再学習

---

## Success Definition

推薦成功

```text
興味あり
+
10試合以上使用
+
継続利用したい
```

---

## Failure Definition

```text
興味あり
+
未使用
```

または

```text
使用
+
継続利用したくない
```

---

## Future

Multi-Armed Bandit

Contextual Bandit

Online Learning

導入予定
