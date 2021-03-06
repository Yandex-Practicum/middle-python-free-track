# SQLite

![image](https://pictures.s3.yandex.net/resources/S1.1_1_SQLite_1602105259.png)

Сейчас база фильмов вашей компании хранится в SQLite. Данных пока не очень много, но вам поручили разработать решение для поиска фильмов по этой базе. Вам стало интересно, почему в качестве БД был выбран SQLite. Выяснилось, что компания молодая, продукт надо было выводить клиентам быстро, поэтому наняли фрилансеров, которые и записали все данные в SQLite. К тому же начальство требует сделать поиск как можно скорее, так как люди уходят с сайта из-за его медленной работы. Поэтому вам предстоит разобраться с SQLite и данными, которые в ней хранятся.

Для начала стоит разобраться, что даёт SQLite и где у неё возможные слабые стороны. Также нужно ответить на вопрос, правильно ли была выбрана база данных и схема для хранения фильмов?

SQLite — это встраиваемая реляционная транзакционная база данных. Она хорошо подходит для:

- **Небольших и средних проектов**. SQLite позволяет очень быстро запуститься и сразу проверять работоспособность сайта на клиентах без расходов на поддержание сложной инфраструктуры.
- **Интернета вещей (IoT)**. Так сложилось, что большинство IoT устройств достаточно ограничены в ресурсах. SQLite — наиболее приспособленная к таким условиям база, так как не требует для работы почти никаких ресурсов.
- **Мобильных приложений**. Сейчас большинство людей использует смартфоны в повседневной работе. SQLite повсюду используется для хранения данных приложений, так как потребляет мало ресурсов. Для мобильных приложений это одно из важнейших условий.
- **Гибкого кэша в микросервисах**. Иногда нужно организовать хранение данных в памяти приложения. Чтобы не думать о быстродействии собственных структур данных, можно использовать SQLite.

Само собой, эта база при всех её достоинствах имеет недостатки. SQLite обычно не используют на важных веб-проектах по нескольким причинам:

- **Безопасность.** Нельзя создавать пользователей БД, следовательно, нельзя предоставить ограниченный доступ.
- **Отсутствие репликации.** У БД нет встроенного решения по репликации данных для обеспечения отказоустойчивости.
- **Сложность масштабирования.** Для масштабирования разработчику придётся менять код приложения. В то же время в других СУБД масштабирование решается на стороне инфраструктуры.
- **Внезапные ошибки.** Несмотря на возможность конкурентного чтения и записи, иногда возникает ошибка `database is locked`. Это может приводить к неожиданным сбоям в программе, которые программист должен обработать. О них можно [почитать дополнительно](https://www.sqlite.org/wal.html){target="_blank"} на английском.

Теперь убедимся на практике, что установка и работа с SQLite достаточно простая.

### Установка

[Скачайте готовый бинарник](https://www.sqlite.org/download.html){target="_blank"} или воспользуйтесь любимым пакетным менеджером:

- Ubuntu/Debian → `apt install sqlite3`;
- Centos/Fedora → `yum install sqlite3`;
- OSX → `brew install sqlite3`;
- Windows → запустить установщих с [официального сайта](https://www.sqlite.org/download.html){target="_blank"}. Подробнее об установке и использовании SQLite для Windows можно почитать [в туториале](https://www.sqlitetutorial.net/download-install-sqlite/){target="_blank"} (на английском).

В целом на этом установка и заканчивается. Дальше разберёмся, как быстро подключиться к базе данных внутри SQLite.

## Работа с SQLite из консоли

Для настройки интерфейса откройте или создайте базу данных с помощью команды `sqlite3 -column -header db.sqlite`:

- `-column` — выводит данные в формате колонок.
- `-header` — отображает название колонок при выводе результатов.
- `db.sqlite` — название файла, где хранятся данные.

Вам дали [файл](https://code.s3.yandex.net/middle-python/learning-materials/db.sqlite){target="_blank"} с текущей базой фильмов.
Попав в консоль, вы можете посмотреть список таблиц в БД. Для этого можно: 

- набрать команду `.tables` и посмотреть схему таблицы с помощью команды `.schema <TABLE_NAME>`;
- выполнить SQL-запрос  `SELECT sql FROM sqlite_master where name='<TABLE_NAME>';`.

Чтобы получить полный список команд, выполните команду `.help`.

## Практическая работа

Итак, вы научились устанавливать SQLite и подключаться к базе данных. Тут же прилетела новая задачка от отдела маркетинга: они хотят сделать подборку фильмов с самым продуктивным сценаристом, самым часто встречающимся актёром и, в честь недели датского кино, написать статью про актёров, работавших с мультипликатором Jørgen Lerdam.

Теперь пора узнать, насколько глубока кроличья нора. Вам необходимо поглубже познакомиться со структурой БД и данными, которые там хранятся. Вы решили спросить у коллеги, как устроена база. К сожалению, он всё это время занимался фронтендом и смутно представляет структуру базы. Однако, вам удалось узнать у него некоторые особенности данных в этой БД:

1. Сценаристы (`writers`) хранятся в разном виде. Когда сценарист один, то появляется просто id этого сценариста. Когда же сценаристов много, то во фронт отдаётся массив записей вида `[{"id": "123"}, {"id": "234"}]`. Ваш коллега грешит на данные в колонках `writer` и `writers`, но у него не доходили руки навести в них порядок, так как он постоянно завален другими задачами по проекту.
2. Поступали жалобы на записи вида `'N/A'` вместо обычных `null` в полях, связанных со сценаристами (`writers`), актёрами (`actors`), режиссёрами (`directors`) и описанием фильмов (`plot`). Коллеги были бы вам очень признательны, если эти записи в итоге вы замените на `null`.
3. Также ваш коллега с фронта упомянул, что им приходится самим парсить данные по жанрам (`genre`) и режиссёрам (`directors`).

Собрав все эти данные и получив файл БД, вам предстоит разобраться со схемой данных и постараться найти те данные, которые от вас требует отдел маркетинга.

Чтобы успешно выполнить задание, нужно:

- уметь строить простейшие SQL-запросы,
- уметь читать ER-диаграммы,
- научиться пользоваться библиотекой sqlite3, которая входит в стандартную библиотеку python.

Если вам не хватает знаний по SQL или появились трудности при составлении запросов, можно восполнить пробелы [с помощью англоязычных туториалов](https://www.w3schools.com/sql/){target="_blank"}.

Как программист, уже знающий Python, вы можете изучить библиотеку sqlite3 [по справочнику](https://pyneng.readthedocs.io/ru/latest/book/18_db/sqlite3.html){target="_blank"} и [в подробной статье](https://python-scripts.com/sqlite){target="_blank"}. 

## ER-диаграммы

ER-диаграммы — это способ описания сущностей (`Entity`) и отношений между ними (`Relation`). В частности, их можно использовать для проектирования моделей баз данных. В вашем случае моделирование состоит из того, чтобы выписать таблицы БД (сущности) и типы связей между таблицами (отношения). Типы связей (множественности связей) обозначаются обычно одним из следующих {{вариантов}}[p2f_python_umloption]:

- `1` — любой записи в таблице соответствует одна запись в другой таблице.
- `0..n` — любой записи в таблице соответствует от 0 до n записей в другой таблице.
- `1..n` — любой записи в таблице соответствует от 1 до n записей в другой таблице. Такую запись используют, чтобы подчеркнуть, что для любой записи должна быть хотя бы одна запись в другой таблице.

Теперь осталось научиться читать такие диаграммы. Перед вами пример связи между двумя таблицами: `movie_actors` и `actors`. Обе таблицы включают в себя список полей и их типы. Самое сложное — правильно отрисовывать типы связей.

![image](https://pictures.s3.yandex.net/resources/S1.1_1_SQLite_1597827267.jpg)

Диаграмму на картинке нужно читать следующим образом: любая запись `movie_actors` связана только с одной записью из таблицы `actors`, но любая запись из таблицы `actors` связана с 0 или более записями из таблицы `movie_actors`.

При составлении типов связей вначале нужно зафиксировать первую таблицу и посмотреть, сколько записей из второй таблицы соответствует одной записи первой таблицы. Потом проделываем то же самое со второй таблицей. Таким образом, вы получаете тип для обоих концов связи. На картинке — многим знакомая по Django связь `OneToMany` или `ForeignKey`.

Подробнее про построение и использование ER-диаграмм можно почитать [на Хабре](https://habr.com/ru/post/440556/){target="_blank"}.

## Мини-квиз

Какой из перечисленных рисунков отражает реальную ER-диаграмму в БД SQLite?

1. ![image](https://pictures.s3.yandex.net/resources/Vopros_1_1600253571.jpg)

   ✅ Здесь представлена правильная ER-диаграмма.

2. ![image](https://pictures.s3.yandex.net/resources/Vopros_2_1600253594.jpg)

   ❌  Типы связей проставлены неверно между таблицами `movies` — `movie_actors` и `movie_actors` — `actors`.

3. ![image](https://pictures.s3.yandex.net/resources/Vopros_3_1600253616.jpg)

   ❌ Здесь появилась несуществующая таблица `movie_writers`. Хотя такая схема отражает более правильную структуру БД.

4. ![image](https://pictures.s3.yandex.net/resources/Vopros_4_1600253638.jpg)

   ❌ На этой ER-диаграмме поля в таблице `movies` не соответствуют тому, что в есть БД SQLite.

Но где же ответ на вопрос о правильности выбора базы данных и схемы для вашей базы фильмов? Пойдём следующим путём: обозначим проблемы, которые есть в текущей схеме и оценим их критичность для практических задач.

1. Отсутствие ключевых полей `primary key` у таблицы `movie_actors`. У вас не будет возможности искать записи, а также есть вероятность вставить дублирующие данные.
2. [Денормализованные](https://ru.wikipedia.org/wiki/%D0%94%D0%B5%D0%BD%D0%BE%D1%80%D0%BC%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F){target="_blank"} данные в таблице `movies`. Для некоторых практических ситуаций это может быть полезно: меньше join-склеек между таблицами. Однако, это усложняет обновление данных, например, если была найдена ошибка в каком-то из популярных жанров.
3. Использование документноориентированной структуры в поле `writers`. Этот подход часто можно встретить в таких БД как MongoDB или Elasticsearch. SQLite — реляционная БД, поэтому предполагает иную работу с данными.
4. Сценаристы `writers` связаны с таблицей фильмов как `ManyToMany`, но без промежуточной таблицы. Это очень усложняет дальнейшую склейку (`join`) данных, чтобы, например, отправлять их в Elasticsearch.

Наличие вышеперечисленных проблем наводит на мысль, что всё-таки структура данных в SQLite разработчиками была выбрана неверно.

Теперь рассмотрим вопрос выбора самой базы данных. Есть важные функции, которые должна поддерживать ваша БД:

1. Для небольшого хранилища под фильмы SQLite подходит хорошо. Но если вы хотите спать спокойно, то лучше иметь репликацию данных. Тогда вам не продётся судорожно восстанавливать их, если упадёт основная БД.
2. С течением времени в вашей компании будут появляться новые сервисы, и тогда придётся разграничивать права доступа между ними. Для этого требуется разделение таблиц на схемы и разделение прав доступа на схемы и таблицы.
3. К БД будут обращаться несколько разных сервисов, что заставит разместить её на отдельном ресурсе — виртуальной машине или железном сервере. Для доступа к этой БД потребуется сетевой доступ. 

SQLite плохо справляется с этими тремя пунктами, хотя для старта разработки эта технология вполне подходит. В будущем необходимо подумать о замене SQLite на более подходящее решение.

## Самостоятельное изучение

- [Официальная документация к SQLite](https://www.sqlite.org/index.html){target="_blank"}
- [Отказоустойчивая распределённая реализация SQLite](https://dqlite.io){target="_blank"}
- [Документация к драйверу SQLite для Python](https://docs.python.org/3/library/sqlite3.html){target="_blank"}
- Альтернативные инструменты для работы с SQLite:
    - [DB Browser](https://sqlitebrowser.org){target="_blank"};
    - [Встроенный инструмент в PyCharm](https://www.jetbrains.com/pycharm/guide/tips/create-sqlite-connection/){target="_blank"};
    - [DBeaver](https://dbeaver.io){target="_blank"}.