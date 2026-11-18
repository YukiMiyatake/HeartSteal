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

---

## High Level Architecture

User

↓

Next.js

↓

API

↓

PostgreSQL

├ Users
├ Answers
├ Champion Reviews
├ Recommendation History
└ User Embeddings

---

## Database Tables

### users

```sql
id
name
created_at
```

### champions

```sql
id
name
role
image_url
```

### questions

```sql
id
category
question
```

### champion_reviews

```sql
id
user_id
champion_id
created_at
```

### review_answers

```sql
id
review_id
question_id
score
```

### user_embeddings

```sql
user_id
embedding vector(256)
updated_at
```

### recommendations

```sql
id
user_id
champion_id
score
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
created_at
```

---

## Embedding Strategy

初期段階はLLM不要。

回答値を直接ベクトル化する。

例:

[
mobility,
playmaking,
lane,
teamfight,
pick,
difficulty
]

---

## Similar User Search

pgvector cosine similarity

```sql
SELECT *
FROM user_embeddings
ORDER BY embedding <=> target_embedding
LIMIT 50;
```
