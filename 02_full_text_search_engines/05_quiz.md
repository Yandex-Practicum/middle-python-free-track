# Квиз

1. Выберите из списка одну или несколько БД, которые являются поисковыми движками.
    - ✅ Elasticsearch → Один из лидирующих поисковых движков с возможностью горизонтального масштабирования.
    - MySQL → Функциональность присутствует, но отсутствуют некоторые важные дополнительные функции, например, нечёткий поиск.
    - ✅ ArangoDB → Достаточно новая БД, которая имеет собственный поисковый движок.
    - Sqlite → Простая БД с функцией простого полнотекстового поиска через встроенный модуль FTS.
    - MongoDB → NoSQL БД, которая часто используется для хранения слабоструктурированных данных. При этом у неё есть функции для [полнотекстового поиска](https://docs.mongodb.com/manual/text-search/){target="_blank"}, но она уступает по скорости работы полноценным поисковым движкам.
    - ✅ Sphinx → Один из самых популярных на текущий момент поисковых движков наряду с Elasticsearch и Solr.

2. Почему реляционные БД не подходят для поиска по базе фильмов? Выберете несколько вариантов.
    - ✅ Сложно сделать масштабируемость решений → К сожалению, большинство реляционных БД не предназначено для массовых операций чтения ввиду их архитектуры. Зато они хорошо подходят для хранения информации и выполнения непротиворечивых изменений данных.
    - ✅ Нет функции полнотекстового поиска → Во всех крупных реляционных БД есть полнотекстовый поиск, но возможности его настройки сильно ограничены.
    - ✅ Нужно много оптимизировать запросы → Да, реляционные БД требуют аккуратной работы при поиске по данным. Необходимо построить индексы, проверить запросы — любая ошибка условий в запросах будет сильно замедлять их.
    - Могут хранить не более 1Тб данных → Любая реляционная БД может хранить данные любого размера, но увеличение данных требует бо́льших манипуляций по оптимизации.

3. Чем Elasticsearch выгодно отличается от других поисковых движков для решения задачи по поиску фильмов?
    - ✅ Возможности по масштабированию → У Elasticsearch есть широкие возможности по масштабированию системы: реплики, шарды, кластер серверов. Всё это отлично подходит для сервиса поиска по фильмам.
    - Нетребовательность к ресурсам (CPU, ОЗУ) → Несмотря на то что в версии 7.7 снижено потребление ОЗУ, оно всё ещё значительно нагружает систему.
    - Хорошая обратная совместимость между версиями → К сожалению, в Elasticsearch зачастую нет обратной совместимости между мажорными версиями.
    - ✅ Использование поискового движка Apache Lucene → Использование этого движка выгодно отличает Elasticsearch от Sphinx, так как даёт больше возможностей для полнотекстового поиска.
    - ✅ Удобный интерфейс запросов → Elasticsearch предоставляет RESTful-интерфейс. Это даёт больше возможностей по применению в коде продуктов, чем языки запросов большинства БД, к которым приходится подключаться через специальные драйверы.

4. Выполните запрос к своему кластеру: `http://127.0.0.1:9200/_xpack?categories=build`. Что указано в графе `tagline`?
    - `You Know, for Search` → Нет. Такой таглайн выводится при запросе к корню API.
    - ✅ `You know, for X`  → Верно! Также можно было подсмотреть результат в [документации по ES REST API](https://www.elastic.co/guide/en/elasticsearch/reference/current/info-api.html){target="_blank"}. 
    - `Elasticsearch` → Хорошая попытка! Но это просто название. 
    - `A plugin for the frozen indices functionality` → Это описание плагина frozen-indices. Его можно найти в перечне плагинов `http://127.0.0.1:9200/_nodes/plugins`.