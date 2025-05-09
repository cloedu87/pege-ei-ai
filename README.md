# pege-ei-ai

install prerequisites

```bash

docker compose up -d db

docker compose run --rm --entrypoint "pip install pgai && pip install -U pgai && python -m pgai install -d postgres://postgres:postgres@db:5432/postgres" vectorizer-worker

docker compose up -d
```

create the extension and data table

```bash
docker compose exec -it db psql
```

```sql
CREATE EXTENSION IF NOT EXISTS ai CASCADE;

CREATE TABLE blog (
    id SERIAL PRIMARY KEY,
    title TEXT,
    authors TEXT,
    contents TEXT,
    metadata JSONB
);
```

create some test data

```sql
INSERT INTO blog (title, authors, contents, metadata)
VALUES
('Getting Started with PostgreSQL', 'John Doe', 'PostgreSQL is a powerful, open source object-relational database system...', '{"tags": ["database", "postgresql", "beginner"], "read_time": 5, "published_date": "2024-03-15"}'),

('10 Tips for Effective Blogging', 'Jane Smith, Mike Johnson', 'Blogging can be a great way to share your thoughts and expertise...', '{"tags": ["blogging", "writing", "tips"], "read_time": 8, "published_date": "2024-03-20"}'),

('The Future of Artificial Intelligence', 'Dr. Alan Turing', 'As we look towards the future, artificial intelligence continues to evolve...', '{"tags": ["AI", "technology", "future"], "read_time": 12, "published_date": "2024-04-01"}'),

('Healthy Eating Habits for Busy Professionals', 'Samantha Lee', 'Maintaining a healthy diet can be challenging for busy professionals...', '{"tags": ["health", "nutrition", "lifestyle"], "read_time": 6, "published_date": "2024-04-05"}'),

('Introduction to Cloud Computing', 'Chris Anderson', 'Cloud computing has revolutionized the way businesses operate...', '{"tags": ["cloud", "technology", "business"], "read_time": 10, "published_date": "2024-04-10"}');
```

create the vectorizer

```sql
SELECT ai.create_vectorizer(
     'blog'::regclass,
     loading => ai.loading_column('contents'),
     embedding => ai.embedding_ollama('nomic-embed-text', 768),
     destination => ai.destination_table('blog_contents_embeddings')
);
```

check vectorizers log: You see the vectorizer worker pick up the table and process it.

```sql
docker compose logs -f vectorizer-worker
```

perform a semantic search: Run the following search query to retrieve the embeddings:

```sql
SELECT
    chunk,
    embedding <=>  ai.ollama_embed('nomic-embed-text', 'good food', host => 'http://ollama:11434') as distance
FROM blog_contents_embeddings
ORDER BY distance
LIMIT 10;

```

Pull the models we need to run the queries and vectorize the data

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
