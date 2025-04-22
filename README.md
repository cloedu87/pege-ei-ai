# pege-ei-ai

## setup whole database stuff

https://github.com/timescale/pgai?tab=readme-ov-file

create the table

```sql
CREATE TABLE wiki (
  id serial PRIMARY KEY,
  url TEXT,
  title TEXT,
  text TEXT
);
```

## load dataset from huggingface

load data from huggingface directly to the database
https://github.com/timescale/pgai/blob/main/docs/utils/load_dataset_from_huggingface.md

## create the vectorizer

```sql
SELECT ai.create_vectorizer(
  'wiki'::regclass,
  'wiki_embeddings',
  '{
    "config_type": "embedding",
    "type": "ollama",
    "implementation": "ollama",
    "model": "all-minilm",
    "dimensions": 384
  }'::jsonb,
  '{
    "config_type": "chunking",
    "implementation": "character_text_splitter",
    "chunk_column": "text",
    "chunk_size": 1000,
    "chunk_overlap": 200,
    "separator": "\\n\\n",
    "is_separator_regex": "false"
  }'::jsonb
);
```

## check for pending items to be vectorized

```sql
-- gets the EXACT count of items pending, but is slow
-- use the ID of the vecorizer you get from SELECT * FROM ai.vectorizer ORDER BY id AS LIMIT 100
select ai.vectorizer_queue_pending(1, exact_count=>true);

-- gets the max int value of 9223372036854775807 if amount of items is above 10000 for performance reasons, but is fast
SELECT * FROM ai.vectorizer_status;
```

## delete an already existing vectorizer

```sql
delete from ai.vectorizer v
    where v.id operator(pg_catalog.=) id-of-vectorizer;
```
