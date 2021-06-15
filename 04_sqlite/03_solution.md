# Авторское решение

```sql
SELECT a.name
FROM movies m
INNER JOIN movie_actors ma ON ma.movie_id=m.id
INNER JOIN actors a ON ma.actor_id=a.id
WHERE m.director LIKE '%Jørgen Lerdam%';
```

```python
# Решение сделано на Python, так как расширение JSON1 для sqlite3 может отсутствовать.

import sqlite3
import json

from collections import Counter

conn = sqlite3.connect('db.sqlite')

counter = Counter()
for row in conn.execute('select writer, writers from movies'):
    writer = row[0]
    if writer:
        counter[writer] += 1
    else:
        writers = json.loads(row[1])
        for w in writers:
            counter[w['id']] += 1

writers_ids = [writer_id for writer_id, _ in counter.most_common()[:2]]
in_placeholders = ','.join(['?'] * len(writers_ids))

row = conn.execute(
    f"select name from writers where id in ({in_placeholders}) and name != 'N/A' limit 1", 
    writers_ids,
).fetchone()

print('Most productive writer:', row[0])
```

```sql
SELECT count(actors.id) cnt,
       actors.name
FROM actors
INNER JOIN movie_actors ON (movie_actors.actor_id = actors.id)
WHERE actors.name != 'N/A'
GROUP BY actors.id
ORDER BY cnt DESC limit 1;
```