# Спецификация методов API

Elasticsearch поднят и заполнен данными. Фронтендеры ждут от вас указаний, как им получать от бэкенда информацию о фильмах. Дело за малым — договориться о том, какие именно данные вы будете передавать и как.

Фронтенд-команде очень не хочется переписывать свой уже работающий код, поэтому они дали вам файл в формате [Swagger/OpenAPI](https://swagger.io/docs/specification/about/){target="_blank"}. В нём описана схема запросов, которая сейчас используется на сайте — попросили сделать точно так же.

Давайте научимся читать эту схему. Для удобного просмотра спецификации [откройте ссылку](https://editor.swagger.io){target="_blank"} и вставьте в редактор содержимое `yaml`-файла.

```yaml
openapi: 3.0.1
info:
  title: Spec
  version: 1.0.0

servers:
- url: https://localhost/api
- url: http://localhost/api
tags:
- name: movies
  description: Всё о фильмах
paths:
  /movies:
    get:
      tags:
      - movies
      summary: Список фильмов
      operationId: listMovies
      parameters:
      - name: limit
        in: query
        description: количество объектов, которое надо вывести
        schema:
          type: integer
          default: 50
      - name: page
        in: query
        description: номер страницы
        schema:
          type: integer
          default: 1
      - name: sort
        in: query
        description: свойство, по которому нужно отсортировать результат
        schema:
          type: string
          default: id
          enum:
          - id
          - title
          - imdb_rating
      - name: sort_order
        in: query
        description: порядок сортировки
        schema:
          type: string
          default: asc
          enum:
          - asc
          - desc
      - name: search
        in: query
        description: "неточный поиск по названию, описанию, актёрам, сценаристам и\
          \ режиссёрам фильма\nПредставьте, что вы вбили в поиск Яндекса \"Звёздны\
          е войны\" или \"Джордж Лукас\" или \"Лукас войны\"  вам выводятся соотве\
          тствующие фильмы. "
        schema:
          type: string
      responses:
        200:
          description: ""
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/ShortMovie'
        400:
          description: "неправильный формат тела запроса"
        422:
          description: "неправильное тело запроса"
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ValidationError'
  /movies/{movieID}:
    get:
      tags:
      - movies
      summary: Получить фильм
      description: Получить фильм
      operationId: getMovieByID
      parameters:
      - name: movieID
        in: path
        required: true
        schema:
          type: string
      responses:
        200:
          description: ""
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Movie'
        404:
          description: Фильм не найден
          content: {}
components:
  schemas:
    ShortMovie:
      required:
      - id
      - title
      type: object
      properties:
        id:
          type: string
        title:
          type: string
        imdb_rating:
          type: number
          format: float
    Writer:
      required:
      - id
      - name
      type: object
      properties:
        id:
          type: string
        name:
          type: string
    Actor:
      required:
      - id
      - name
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
    Movie:
      required:
      - id
      - title
      type: object
      properties:
        id:
          type: string
        title:
          type: string
        description:
          type: string
        imdb_rating:
          type: number
          format: float
        writers:
          type: array
          items:
            $ref: '#/components/schemas/Writer'
        actors:
          type: array
          items:
            $ref: '#/components/schemas/Actor'
        genre:
          type: array
          items:
            type: string
        director:
          type: array
          items:
            type: string
    ValidationError:
      type: object
      properties:
        detail:
          type: array
          items:
            type: object
            properties:
              loc:
                type: array
                example: ["query", "limit"]
                items:
                  type: string
              msg:
                example: "value is not a valid integer"
                type: string
              type:
                type: string
                example: "type_error.integer"
```

OpenAPI используется для описания методов REST API и данных, которые эти методы принимают и возвращают.

Вверху страницы указан базовый URL вашего сервиса.

![image](https://pictures.s3.yandex.net/resources/S1.1_1_Flask_1594165910.png)

Под ним расположен список `end-points`. 

В URL обычно закладывается идея того, что делает метод. В примере прослеживается следующая логика: есть фильмы, по которым можно получить данные. Подобный формат API называется REST. О нём подробно написано на сайте [документации Microsoft](https://docs.microsoft.com/ru-ru/azure/architecture/best-practices/api-design){target="_blank"}.

В левой части списка указан HTTP-метод, с которым можно обратиться по этому URL. Справа от него — сам URL. Первый метод ждёт GET-запрос по адресу на `http://localhost:8000/movies`, а второй — GET-запрос к адресу `http://localhost:8000/movies/{movieID}`. Если обратится к ним, используя, например, метод `POST` или `DELETE`, сервер вернёт код ответа `405` — Method Not Allowed.

![image](https://pictures.s3.yandex.net/resources/S1.1_2_Flask_1594165913.png)

Нажмите на интересующий вас метод, чтобы раскрыть подробную информацию о нём. Первый раздел — `Parameters` — описывает список параметров, которые можно передать методу.

1. Обязательные параметры будут отмечены звёздочкой.
2. Название параметра. Конкретный формат данных, который ожидает сервис.
3. Тип данных. Если сервису передают строку вместо числа, то он должен вернуть ошибку. Например, если значение `limit` будет «абракадабра», то сервис вернёт ошибку валидации.
4. Передача параметра. Показывает, каким образом передаётся параметр — `query`, `path` или `body`. 
    - Если указан `query`, то параметр передаётся через `URL`, например, `http://localhost:8000/api/movies?limit=42`.
    - Если указан `path`, то параметр передаётся непосредственно как часть `URL`. Пример можно посмотреть в спецификации метода для получения фильма. Вместо `{movie_id}` нужно указать идентификатор фильма. 
    - Если указан `body`, то параметр можно передать при использовании HTTP-методов `POST`, `PUT`, `PATCH`. Если поиск фильма вызывался с помощью `POST`-запроса, то вы бы передали `JSON {"limit": 42}`.

![image](https://pictures.s3.yandex.net/resources/S1.1_4_Flask_1594165917.png)

За разделом `Parameters` следует `Responses`. Задача этого раздела — показать формат ответа сервера. Спецификация покажет, какие атрибуты вернутся, а также какие типы данных и какие поля будут обязательными. Для каждого HTTP-кода ответа отображается пример возвращаемых данных.

1. Responses описывает, какие данные может вернуть метод. Помните, что в примере могут быть данные, не имеющие ничего общего с реальностью.
2. Пример ответа, если во время работы метода не произошло никаких ошибок. 
3. Пример ответа, который вернул ошибку валидации запроса. Пример показывает, какой `JSON` вернётся, если клиент передал неправильный тип данных в `limit`, а также то, что возвращается `422` HTTP-статус.

![image](https://pictures.s3.yandex.net/resources/S1.1_5_Flask_1594165920.png)

Чтобы вместо примера увидеть подробное описание возвращаемых данных, нажмите на `Schema`. Откроется вкладка с перечнем типов данных для каждого атрибута, их описанием и красными звёздочками напротив полей, которые никогда не будут пустыми.

![image](https://pictures.s3.yandex.net/resources/S1.1_6_Flask_1594165922.png)

Обратите внимание на атрибут `imdb_rating`: для него указан формат `float`. Формат `json` не умеет отличать целые числа от дробных. Любое число для него — `number`. Чтобы можно было точно указывать, какое значение ожидает сервис API, в OpenAPI добавили немного более подробные типы.

Нажав на кнопку `Try it out`, вы сможете отправить запрос на свой сервер.

![image](https://pictures.s3.yandex.net/resources/S1.1_7_Flask_1594165925.png)

После нажатия откроется форма для ввода значений.

1. В полях можно указывать любые значения. Swagger будет следить за тем, чтобы тип данных, которые вы отправляете, совпадал с описанным в схеме.
2. Когда вы нажмёте кнопку `Execute`, запрос отправится на сервер.

![image](https://pictures.s3.yandex.net/resources/S1.1_8_Flask_1594165928.png)

Когда сервер вернёт вам ответ, он отобразится на странице, а также сгенерируется `curl`-команда для выполнения запроса из консоли.

1. После нажатия кнопки `Execute` ...
2. ...вы увидите данные, с которыми был вызван метод.
3. И ответ, полученный от сервиса.

![image](https://pictures.s3.yandex.net/resources/S1.1_9_Flask_1594165943.png)

Сейчас на вашем `localhost` не запущен сервис, который мог бы ответить Swagger, поэтому после нажатия на кнопку вы увидите надпись `TypeError: Failed to fetch`. 


## Квиз

1. Какие поля в ответе метода `GET /movies/{movieID}` никогда не окажутся пустыми?
    - `imdb_rating` → Это поле необязательно. 
    - ✅`title` → Да. Мы можем рассчитывать, что каждого фильма точно будет название.
    - `description` → К сожалению, описание фильма может отсутствовать.
    - ✅`id` → Верно! Кстати, это тот же Primary Key фильма, который подписан в `URL` как `movieID`.
    - `genre` → Схема подсказывает, что жанр фильма может быть не указан в БД.
    
2. Какой параметр у метода `GET /movies` указывает на поле, по которому будет отсортирован результат?
    - `page` → Нет. Здесь указывается номер страницы для пагинации.
    - ✅`sort` → Именно так. 
    - `sort_order` → Этим параметром можно отсортировать список по возрастанию или убыванию.
    
3. Сколько объектов вернёт запрос к методу `GET /movies`, если в БД находится 1500 записей?
    - ✅50  → У параметра `limit`, отвечающего за количество элементов на странице, именно такое значение по умолчанию.
    - 100 → Мимо!
    - 1500 → Иногда API проектируют так, чтобы возвращались все записи. Но не в этом случае.
    - ✅110010 → В принципе верно, если вы мыслите в двоичной системе счисления :)

## О пользе Swagger

Swagger полезен, если вы пишете приложение, которое должно взаимодействовать с другой системой. Его содержимое может заменить документацию.

Кроме описания схем вручную, он поддерживает автогенерацию.  Например, если в вашем сервисе используется [Django Rest Framework](https://www.django-rest-framework.org/){target="_blank"}, то библиотека [drf-yasg](https://drf-yasg.readthedocs.io/en/stable/readme.html){target="_blank"}
самостоятельно создаст акутальное описание всего вашего API. Оно будет меняться автоматически при каждом обновлении API.

Если у вашего API есть спецификация, то сервисы, которые хотят с ним взаимодействовать, могут с помощью [swagger-codegen](https://github.com/swagger-api/swagger-codegen){target="_blank"} сгерерировать код для любого популярного языка. Модуль автоматически создаст нужные структуры, 
будет проверять, что передаются данные нужных типов и уменьшит количество кода, который придётся написать вручную.