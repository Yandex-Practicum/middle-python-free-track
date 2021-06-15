# Получение данных из Elasticsearch

Вы уже познакомились с загрузкой данных и простейшими запросами на получение данных с использованием полнотекстового поиска. Теперь пора применить знания на практике.

Для реализации API нужно познакомиться поближе с популярными видами полнотекстовых запросов.

- Запросы по совпадениям (`match query`) — самый распространённый вид запросов для полнотекстового поиска.
Позволяет искать по указанной фразе с предварительным анализом содержимого запроса.
Этот вид запросов подходит для вашего сервиса поиска по фильмам.
Подробнее про механизм работы можно прочитать [в документации на английском](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html){target="_blank"}.
- Запросы по множественному совпадению (`multi_match query`) — надстройка над обычным match query, чтобы проводить поиск по нескольким полям одновременно. В вашем случае этот вид запросов наиболее подходящий.
Для дополнительной информации читайте [документацию на английском](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html){target="_blank"}.
- Запросы через интервалы (`intervals query`) комбинируют различные группы условий, например, для поиска фраз, стоящих в определённом порядке. В вашем случае такие сложные запросы не нужны.
Подробнее о них можно прочитать [в документации на английском](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-intervals-query.html){target="_blank"}.
- Запросы с использованием языка запросов Apache Lucene (`query string query`) — гибкий способ строить запросы, но и самый сложный для отладки и написания.
Этот вид запросов позволяет низкоуровнево описать поисковый запрос и настроить его под определённую задачу.
Все подробности — [в документации на английском](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html){target="_blank"}.

Есть и [другие типы запросов](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html){target="_blank"} для поиска по тексту. Они предназначены для более точной (экспертной) настройки поиска в разных частях приложения, например, в поиске по справочнику, в механизме автодополнения или в расширенном (более точном) поиске. Мы сейчас сосредоточимся на стандартных типах запросов – они вполне подойдут для наших целей.

## Использование multi_match query

Перейдём к написанию запросов для поиска по фильмам и описаниям.
Воспользуемся для этого `multi_match query`:

```bash
curl http://127.0.0.1:9200/movies/_search -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "campfir",
      "fields": [
        "title",
        "description",
        "genre",
        "actors_names",
        "writers_names",
        "director"
      ]
    }
  }
}'
```

После выполнения этого запроса, увы, ничего не найдётся, потому что Elasticsearch пытается найти прямые совпадения по словам. Слова `campfir` нет в ваших данных.

Чтобы починить поиск по неполному слову, стоит добавить к запросу [свойство](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html){target="_blank"} `[fuzziness]`.

```bash
curl http://127.0.0.1:9200/movies/_search -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "campfir",
      "fuzziness": "auto",
      "fields": [
        "title",
        "description",
        "genre",
        "actors_names",
        "writers_names",
        "director"
      ]
    }
  }
}'
```

В ответе вы получите список фильмов из одного элемента — `"Star Wars Forces of Destiny: Volume 2"`.

```json
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 31.628067,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "tt7589570",
        "_score" : 31.628067,
        "_source" : {
          "id" : "tt7589570",
          "genre" : [
            "Animation",
            "Adventure",
            "Family"
          ],
          "writers" : [
            {
              "id" : "0b60f2f3e32cc50b4bde3b30e67e55bb51f59523",
              "name" : "Jennifer Muro"
            }
          ],
          "actors" : [
            {
              "id" : "104",
              "name" : "Anthony Daniels"
            },
            {
              "id" : "497",
              "name" : "David Acord"
            },
            {
              "id" : "51",
              "name" : "Ashley Eckstein"
            },
            {
              "id" : "53",
              "name" : "Dee Bradley Baker"
            }
          ],
          "actors_names" : [
            "Anthony Daniels",
            "David Acord",
            "Ashley Eckstein",
            "Dee Bradley Baker"
          ],
          "writers_names" : [
            "Jennifer Muro"
          ],
          "imdb_rating" : 6.4,
          "title" : "Star Wars Forces of Destiny: Volume 2",
          "director" : [
            "Brad Rau"
          ],
          "description" : "While cooking stew over her campfire, Maz tells more tales of courage."
        }
      }
    ]
  }
}
```

Однако со свойством `fuzziness` стоит быть осторожным: оно может выдать слишком много нерелевантных результатов.

Если вы захотите найти фильм по каким-то общим словам, то результат может не совпасть с ожиданиями.
Например, передав `«neubauer star»` в параметр `search`, вы найдёте фильм `Star Raiders`.
Так получается из-за неточного поиска по полям названию, описанию, актёрам, сценаристам и режиссёрам.

Чтобы избежать этого, можно воспользоваться настройкой весов у полей.

Веса указываются после названия поля. Запись `genre^3` означает, что вес поля «жанр» — 3.
Это одна из множества настроек ElasticSearch, которой можно повлиять на {{релевантность}}[p2f_python_relevance] результатов.

Например, если совпадение в названии более важно, чем в описании, можно задать названию больший вес:
`title^5, description^4, director`. При такой настройке вклад поля `title` будет в 5 раз больше, чем у поля `director`.

## Квиз

1. Выполните запрос. Какой документ вернет по нему Elasticsearch?

    ```bash
    curl -XPUT http://127.0.0.1:9200/movies/_search -H 'Content-Type: application/json' -d '{
      "query": {
        "multi_match": {
          "query": "mystery",
          "fuzziness": "auto",
          "fields": [
            "title^2",
            "description",
            "genre",
            "actors_names",
            "writers_names",
            "director"
          ]
        }
      },
      "size": 1
    }'
    ```

    - ✅ Star Wars: The Magic & the Mystery → Правильно. В запросе у названия задан наибольший вес, этот фильм имеет точное совпадение по слову mystery в названии и, следовательно, наиболее релевантен запросу.
    - Shuten Doji: The Star Hand Kid 2 - Demon Battle in the Firefly Field → Неправильно. В названии фильма нет слова mystery!
    - Star Trek: The Next Generation → Неправильно. Также нет совпадений по названию.

2. Выполните запрос. Какой документ вернет по нему Elasticsearch?

    ```bash
    curl -XPUT http://127.0.0.1:9200/movies/_search -H 'Content-Type: application/json' -d '{
      "query": {
        "multi_match": {
          "query": "mystery",
          "fuzziness": "auto",
          "fields": [
            "title",
            "description^2",
            "genre",
            "actors_names",
            "writers_names",
            "director"
          ]
        }
      },
      "size": 1
    }'
    ```

    - Star Wars: The Magic & the Mystery → Неправильно. В описании фильма не встречается слово mystery, значит, документ будет располагаться ниже в выдаче.
    - ✅ Shuten Doji: The Star Hand Kid 2 - Demon Battle in the Firefly Field → Правильно. В запросе у описания задан наибольший вес, этот фильм имеет точное совпадение по слову mystery в описании и, следовательно, наиболее релевантен запросу.
    - Star Trek: The Next Generation → Неправильно. В описании фильма не встречается слово mystery, значит, документ будет располагаться ниже в выдаче.

3. Выполните запрос. Какой документ вернет по нему Elasticsearch?

    ```bash
    curl -XPUT http://127.0.0.1:9200/movies/_search -H 'Content-Type: application/json' -d '{
      "query": {
        "multi_match": {
          "query": "mystery",
          "fuzziness": "auto",
          "fields": [
            "title",
            "description",
            "genre^2",
            "actors_names",
            "writers_names",
            "director"
          ]
        }
      },
      "size": 1
    }'
    ```

    - Star Wars: The Magic & the Mystery → Неправильно. Жанр этого фильма: Documentary, фильм будет располагаться ниже в выдаче.
    - Shuten Doji: The Star Hand Kid 2 - Demon Battle in the Firefly Field → Неправильно. Жанры этого фильма: Animation, Fantasy и Horror, фильм будет располагаться ниже в выдаче.
    - ✅ Star Trek: The Next Generation → Правильно. В запросе у жанра задан наибольший вес, этот фильм имеет точное совпадение по слову Mystery в списке жанров и, следовательно, наиболее релевантен запросу.
