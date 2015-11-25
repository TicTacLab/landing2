---
title: SettlementEngine. Руководство по интеграции. 
---
## SettlementEngine

# Руководство по интеграции 

## Введение

В данном документе описан порядок работы с SettlementEngine REST API. Предполагается, что Excel файл уже [подготовлен](/ru/doc/sengine/file-preparation/) и загружен посредством [Административного интерфейса](/ru/doc/user-guide/).

## Использование SettlementEngine REST API 

Использование SettlementEngine REST API состоит из нескольких шагов:

1. [Создание события](#event-create)
2. [Заполнение журнала события](#log-fill-in)
3. [Освобождение ресурсов](#release)

{:#event-create}
### Создание события

Для накопления событий в журнале матча, SEngine виртуально создает копию Excel файла для каждого матча. Каждый матч в SEngine использует два идентификатора:

* event-id
* file-id

Ресурсы выделяются после вызова метода *CREATE-EVENT*.

{:#log-fill-in}
### Заполнение журнала события

Для расчета маркетов, SettlementEngine использует на журнал события. Заполнение журнала события происходит с помощью методов *APPEND-EVENT-LOG* и *SET-EVENT-LOG*. Эти методы возвращают маркеты рассчитанные на основе нового журнала события.

Подробно о подготовке файла для расчета маркетов описано в [руководстве](/ru/doc/sengine/file-preparation/).

Для добавления данных в журнал события, запросы должны отправляться последовательно. SettlementEngine блокирует запросы до тех пор, пока не закончится выполнение предыдущего запроса. Попытка вызова методов API, у заблокированного события, приводит к ошибке  **423 "Calculation in progress"**.

{:#release}
### Освобождение ресурсов
После окончания события следует освободить вычислительные ресурсы вызвав метод *FINALIZE-EVENT*.

## SettlementEngine REST API

### Введение

Любой запрос может вернуть ошибку **500 "ISE"** в случае внутренних ошибок системы. Если сервис не доступен возвращается ошибка **503 "STU"**. Все ответы REST API имеют Content-Type: "application/json".

**Структура ответа**

~~~ javascript
{
  "status": 200,
  "data": null // или {} или [] в зависимости от запроса 
}
~~~

**Структура ошибки**

~~~ json
{
  "status": 404,
  "errors": 
  [
    {
      "code": "ENF",
      "message": "Event not found"
    }
  ]
}
~~~

### Method: CREATE-EVENT

**POST /events**

Создает уникальное событие для работы с сервисом.


**Parameters**

* file-id - обязательно, integer, ID Excel файла по которому будет проводится событие
* event-id - обязательно, string, ID события которое будет проводиться
* description - обязательно, string, строковое описание события
* filters - опционально, hashmap, должен содержать названия столбцов и список значений из вкладки *OUT*. Очищает вкладку *OUT* от всех не указанных значений.

**Request body**

~~~ json
{
  "params": {
    "file-id": 12,
    "event-id": "550e8400-e29b-41d4-a716-446655440000"
    "description": "Soccer Dinamo-Kiev"
    "filters" : {
      "Market name" : [ "MATCH_BETTING", "MATCH_DRAW_NO_BET" ]
    }
  }
}
~~~

**Responses**

* 200 - успешное создание события
* 400 "SLE" - лимит событий/сессий превышен
* 400 "EAC" - если событие уже создано
* 400 "MFP" - параметры file-id или event-id не верны
* 404 "FNF" - файл не найден

**Request example** <a href="#curl">*<a/>

~~~
curl -X POST -d <<json-request-body>> http://settlement.engine/events
~~~

**Response 200 data example**

~~~
response data field will be null
~~~

### Method: FINALIZE-EVENT

**DELETE /events/:event-id**

Завершает события ранее созданное с помощью CREATE-EVENT, освобождает вычислительные ресурсы выделенные под событие.

**Parameters**

* event-id - обязательно, string, ID события

**Responses**

* 200 - успешное удаление события
* 404 "ENF" - если событие не существует

**Request example** <a href="#curl">*<a/>

~~~
curl -X DELETE http://settlement.engine/events/550e8400-e29b-41d4-a716-446655440000
~~~

**Response 200 data example**

~~~
response data field will be null
~~~

### Method: GET-SETTLEMENTS

**GET /events/:event-id/settlements**

Возвращает текущее состояние вкладки *OUT*.

**Parameters**

* event-id - обязательно, string, ID события

**Responses**

* 200 - успешное получение расчета маркетов
* 400 "MFP" - если event-id или request body не верного формата
* 404 "ENF" - если событие не существует

**Request example** <a href="#curl">*<a/>

~~~
curl http://settlement.engine/events/550e8400-e29b-41d4-a716-446655440000/settlements
~~~

**Response 200 data example**

~~~ json
[ {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "HOME",
  "Calc" : "lose"
}, {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "DRAW",
  "Calc" : "win"
}, {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "AWAY",
  "Calc" : "lose"
} ]  
~~~

### Method: GET-EVENT-LOG

**GET /events/:event-id/event-log**

Возвращает текущее состояние журнала события из вкладки *EventLog*.

**Parameters**

* event-id - обязательно, string, ID события

**Responses**

* 200 - успешное получение журнала события
* 400 "MFP" - если event-id не верного формата
* 404 "ENF" - если событие не существует

**Request example** <a href="#curl">*<a/>

~~~
curl http://settlement.engine/events/550e8400-e29b-41d4-a716-446655440000/event-log
~~~

**Response 200 data example**

~~~ json
[ {
  "EventType" : "Goal",
  "Team" : "Team1",
  "GamePart" : "Half1"
}, {
  "EventType" : "Goal",
  "Team" : "Team2",
  "GamePart" : "Half1"
} ]  
~~~

### Method: APPEND-EVENT-LOG

**POST /events/:event-id/event-log/append**

Добавляет записи в конец журнала события во вкладке *EventLog*.

**Parameters**

* event-id - обязательно, string, ID события

**Request body**

~~~ json
{
  "params": [
    {
      "EventType" : "Goal",
      "Team" : "Team1"
    }, {
      "EventType" : "Goal",
      "Team" : "Team2"
    }
  ]
}
~~~

**Responses**

* 200 - успешное добавление записей в журнал события. Тело ответа содержит расчет маркетов
* 400 "MFP" - если event-id или request body не верного формата
* 404 "ENF" - если событие не существует

**Request example** <a href="#curl">*<a/>

~~~
curl http://settlement.engine/events/550e8400-e29b-41d4-a716-446655440000/event-log/append
~~~

**Response 200 data example**

~~~ json
[ {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "HOME",
  "Calc" : "lose"
}, {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "DRAW",
  "Calc" : "win"
}, {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "AWAY",
  "Calc" : "lose"
} ]  
~~~

### Method: SET-EVENT-LOG

**POST /events/:event-id/event-log/set**

**Очищает** журнал события во вкладке *EventLog* и вставляет в него новые записи.

**Parameters**

* event-id - обязательно, string, ID события

**Request body**

~~~ json
{
  "params": [
    {
      "EventType" : "Goal",
      "Team" : "Team1"
    }, {
      "EventType" : "Goal",
      "Team" : "Team2"
    }
  ]
}
~~~

**Responses**

* 200 - успешное добавление записей в журнал события. Тело ответа содержит расчет маркетов
* 400 "MFP" - если event-id или request body не верного формата
* 404 "ENF" - если событие не существует

**Request example** <a href="#curl">*<a/>

~~~
curl http://settlement.engine/events/550e8400-e29b-41d4-a716-446655440000/event-log/set
~~~

**Response 200 data example**

~~~ json
[ {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "HOME",
  "Calc" : "lose"
}, {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "DRAW",
  "Calc" : "win"
}, {
  "Market name" : "MATCH_BETTING",
  "Outcome" : "AWAY",
  "Calc" : "lose"
} ]  
~~~



## Ваш сервис готов!

Теперь Вы можете проводить свои спортивные события с помощью SettlementEngine API и Ваших алгоритмов для расчета маркетов.

<hr />

<blockquote class="blockquote-reverse">
  <p>Мы много работаем над улучшением нашего сервиса, и всегда будем рады любому отзыву о его работе.</p>
  <footer>С уважением, комманда BetEngines</footer>
</blockquote>

<hr />

{:#curl}
\* Для примеров используется утилита *curl*. Вариант **curl** для своей системы можно [скачать здесь](http://curl.haxx.se/download.html).
