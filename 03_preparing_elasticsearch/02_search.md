# Поиск по данным

Процесс работы поискового движка делится на два этапа:

* Индексирование, с которым вы познакомились на прошлом уроке.
* Поиск по индексу, с которым вы познакомитесь прямо сейчас.

Общий алгоритм поиска выглядит так:

**(Схема)** 

![scheme](pictures/2_search/scheme.png)

1. Получили поисковый запрос от клиента.
2. Обработали поисковый запрос для запроса в индекс.
3. Достаём из индекса данные, соответствующие запросам.
4. Рассчитываем релевантность результатов.
5. Ранжируем результаты запроса.
6. Отдаём результаты клиенту.

P.S. Некоторые шаги этой схемы могут быть объединены в зависимости от использования тех или иных алгоритмов.

Про обработку данных поговорим в следующем уроке. А пока разберёмся с ранжированием — сортировкой результатов поиска с учётом соответствия поисковому запросу.

Поисковый движок должен уметь ранжировать результаты поиска. Иначе как понять, какие результаты наиболее соответствуют поисковому запросу. 
Релевантность документа — это оценка соответствия результата поисковому запросу. Это число, которое вычисляется в зависимости от используемых поисковых алгоритмов. В простом смысле ранжирование — это сортировка результатов по релевантности.

Посмотрим на примере. Типовой запрос на получение всех данных в конкретном индексе будет выглядеть следующим образом:

```bash
curl -XGET http://127.0.0.1:9200/table/_search
```

Вы получите ответ:

```json
{
	"took": 74,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 1,
			"relation": "eq"
		},
		"max_score": 1.0,
		"hits":[
			{
				"_index": "table",
				"_type": "_doc",
				"_id": "nZElhnIB-W6dcc_UKo2E",
				"_score": 1.0,
				"_source": {
					"text_field": "my pretty text",
					"number": 16
				}
			}
		]
	}
}
```

Вернулось много информации. Прежде всего обратите внимание на параметр `hits.hits` — он содержит ответ на ваш запрос. Учтите, что по умолчанию такой запрос отдаёт только 20 записей.

Определимся с некоторыми параметрами ответа:

- `took` — говорит, сколько времени занял запрос в миллисекундах.
- `hits.total.value` — говорит о том, сколько всего значений находится в индексе.
- `hits.hits._score` — релевантность документа поисковому запросу. То самое число, по которому происходит ранжирование. 
- `hits.hits._source` — говорит, какие данные содержит запись.

Остальные поля в контексте решаемой задачи не должны сильно вас интересовать. Если вам хочется углубить свои знания по теме, прочтите [документацию Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#docs-get-api-response-body){target="_blank"} на английском.

Для получения одной записи добавьте к запросу id документа:

```bash
curl -XGET http://127.0.0.1:9200/table/_doc/nZElhnIB-W6dcc_UKo2E
```

Получите ответ:

```json
{
	"_index": "table",
	"_type": "_doc",
	"_id": "nZElhnIB-W6dcc_UKo2E",
	"_version": 2,
	"_seq_no": 1,
	"_primary_term": 1,
	"found": true,
	"_source": {
		"text_field": "my pretty text",
		"number": 16
	}
}
```

Здесь всё просто: вы получили информацию о конкретной записи, примерно такую же, как и при сохранении. Попробуйте послать запрос с несуществующим `id`, чтобы убедиться, что Elasticsearch отработает его в штатном режиме.

Перейдём к более интересной части. До этого момента вы просто посылали запросы на получение всех записей и записей по конкретному `id`. Теперь попробуем сделать поиск через фильтрацию.

Предположим, что в индексе `table` прибавилось данных, и теперь вам нужно найти все записи, у которых `number` больше 10. В SQL этот запрос выглядит так:

```sql
SELECT * FROM "table" WHERE number > 10;
```

Такой же запрос на классическом языке запросов Elasticsearch выглядит следующим образом:

```bash
curl -XGET http://127.0.0.1:9200/table/_search -H 'Content-Type: application/json' -d'
{
	"query": {
		"bool": {
			"filter": [
				{
					"range": {
						"number": {
							"gt": 10
						}
					}
				}
			]
		}
	}
}'
```

Смотрится тяжеловато, но со временем к такому виду запросов можно привыкнуть.

Для тех, кому хочется привычного SQL в рамках Elasticsearch, посмотрите в сторону SQL-синтаксиса, который появился совсем недавно. 
В таком синтаксисе ваш запрос будет выглядеть следующим образом:

```bash
#table дан в кавычках, потому что это служебное слово для SQL-запросов в Elasticsearch

curl -XPOST http://127.0.0.1:9200/_sql -H 'Content-Type: application/json' -d'
{
	"query": "SELECT * FROM \"table\" WHERE number > 10"
}'
```

В результате получается:

```json
{
	"columns": [
		{
			"name": "number",
			"type": "long"
		},
		{
			"name": "text_field",
			"type": "keyword"
		}
	],
	"rows":[
		[16, "my pretty text"],
		[17, "my pretty text2"]
	]
}
```

Увы, такой интерфейс очень ограничен в возможностях по сравнению с SQL в реляционных базах данных. По этой причине данную функциональность нужно использовать с особой осторожностью. 

Однако, у такого интерфейса есть другая полезная сторона. Если вам не хочется учить Elasticsearch {{DSL}}[p2f_python_dsl], можно воспользоваться функцией `translate`:

```bash
curl -XPOST http://127.0.0.1:9200/_sql/translate -H 'Content-Type: application/json' -d'
{
	"query": "SELECT * FROM \"table\" WHERE number > 10"
}'
```

Здесь предлагаем вам поэкспериментировать. Посмотрите, насколько больше информации выведет перевод из SQL в Elasticsearch DSL, и сравните с тем запросом, который вы бы написали самостоятельно. 

Спойлер: Elasticsearch DSL всё же придётся подучить, чтобы делать максимально эффективные запросы для ваших задач.

Если всё-таки хочется поиграть с SQL для Elasticsearch, то советуем ознакомиться с [официальной документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-getting-started.html){target="_blank"} на английском. К сожалению, на русском языке информации по Elasticsearch SQL нет.
