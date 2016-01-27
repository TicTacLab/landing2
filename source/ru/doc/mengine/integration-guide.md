---
title: MathEngine. Руководство по интеграции
---
## MathEngine

# Руководство по интеграции 

## Введение

В данном документе описан порядок работы с MathEgnine REST API. Предполагается, что Excel файл уже [подготовлен](/ru/doc/mengine/file-preparation/) и загружен посредством [Административного интерфейса](/ru/doc/user-guide/).

## Использование MathEngine REST API 

Использование MathEngine REST API состоит из нескольких шагов:

1. [Создание события с уникальным](#event-create) *event-id*.
2. [Заполнение значений](#in-fill-in) из вкладки "IN" и отправка их к сервису BetEngines. В ответ BetEngines отправит раcсчитанные значения из вкладки "OUT". Этот шаг повторяется до окончания события.
3. [Освобождение ресурсов](#release) после окончания события. 

А теперь о каждом шаге детальнее. 

{:#event-create}
### Создание события с уникальным *event-id*

Для повышения скорости расчета BetEngines резервирует вычислительные мощности для каждого события. Каждое событие в BetEngines использует два идентификатора:

* event-id
* file-id

Ресурсы выделяются автоматически, после первого вызова API методов *CALCULATE* или *IN-PARAMS*. Ресурсы будут выделены повторно в случае вызова методов *CALCULATE* или *IN-PARMS*, после вызова метода *RELEASE*.

Специального создания события и выделения ресурсов не требуется.

{:#in-fill-in}
### Заполнение значений из вкладки "IN" и получение результата из вкладки "OUT" 

Для расчета модели описанной в Excel файле предназначен метод *CALCULATE*.

<div class="well well-sm">
<b>Важно!</b> Вкладка "IN" должна содержать две обязательных колонки <b>"id"</b> и <b>"value"</b> и любое количество дополнительных колонок.
</div>

Подробно о подготовке файла описано в [Руководстве](/ru/doc/mengine/file-preparation/).

Запросы для расчета события должны происходить последовательно, BetEngines блокирует запросы пока не закончится выполнение предыдущего. Попытка вызова  API у заблокированного события приводит к ошибке  **423 "Calculation in progress"**.

{:#release}
### Освобождение ресурсов
После окончания события следует освободить ресурсы вызвав метод *RELEASE*. Также ресурсы освобождаются автоматически по истечении 15 минут, если к событию не было ни одного обращения за этот период. 

## BetEngines REST API

- [Введение](#api-general)
- [Аутентификация](#api-auth)
- [API](#api-in-params)
  * [IN-PARAMS](#api-in-params)
  * [CALCULATE](#api-calculate)
  * [RELEASE](#api-releases)

{:#api-general}
### Введение

Любой запрос может вернуть ошибку **500 "ISE"** в случае внутренних ошибок системы. Если сервис не доступен возвращается ошибка **503 "STU"**. Все ответы BetEngines имеют Content-Type: "application/json". У ответа 204 нет содержимого, как в спецификации к протоколу HTTP.

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
  "status": 423,
  "errors": 
  [
    {
      "code": "CIP",
      "message": "Calculation in progress"
    }
  ]
}
~~~


{:#api-auth}
### Аутентификация

Math Engine использует HTTP [Basic Аутентификацию](https://www.w3.org/Protocols/HTTP/1.0/spec.html#BasicAA). Следуя описанию Basic Аутентификации, все запросы должны содержать заголовок Authorization с аутентификационными данными, например:

<pre>
GET /api/files/120/72c361803a5b116e0682581fe958fdee/in-params HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: math.engine
Accept-Language: en-us
Accept-Encoding: gzip, deflate
<b>Authorization: Basic <base64 encoded "login:password"></b>
Connection: Keep-Alive
</pre>

Для логина "Aladdin" и пароля "open sesame" base64 форма "Aladdin:open sesame" будет "QWxhZGRpbjpvcGVuIHNlc2FtZQ==". Такой запрос будет иметь следующий вид:

<pre>
GET /api/files/120/72c361803a5b116e0682581fe958fdee/in-params HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: math.engine
Accept-Language: en-us
Accept-Encoding: gzip, deflate
<b>Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==</b>
Connection: Keep-Alive
</pre>


Если заголовок Authorization отсутствует или авторизационные данные не верны Math Engine вернет ошибку **401 "UAR"** с WWW-Authenticate заголовком:

    WWW-Authenticate: Basic realm="mengine"

{:#api-in-params}
### Method: IN-PARAMS

**GET /api/files/$file-id/$event-id/in-params**

Возвращает содержимое вкладки "IN" указанного файла. Сочетание event-id + file-id должно быть уникальным для идентификации события.
Результат этого метода можно использовать для вызова расчета с помощи метода *CALCULATE*.

**Parameters**

* file-id - обязательно, ID Excel файла, по которому будет проводится событие
* event-id - обязательно, уникальный ID события

**Responses**

* 200 - успешное получение параметров
* 400 "SLE" - лимит сессий превышен
* 404 "FNF" - файл не найден

**Request example** <a href="#curl">*<a/>

~~~
curl --header "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" http://math.engine/api/files/120/72c361803a5b116e0682581fe958fdee/in-params
~~~

**Response 200 data example**

~~~ json
{
  "status":200,
  "data":
  [
    {
      "code":"P_SCORE_A_PART_1",
      "value":0.0,
      "name":"Score A Part 1",
      "type":"Parameter",
      "id":1
    }, 
    {
      "code":"P_SCORE_B_PART_1",
      "value":0.0,
      "name":"Score B Part 1",
      "type":"Parameter",
      "id":2
    }, 
    {
      "code":"P_SCORE_A_PART_2",
      "value":0.0,
      "name":"Score A Part 2",
      "type":"Parameter",
      "id":3
    },
    {
      "code":"P_SCORE_B_PART_2","value":0.0,
      "name":"Score B Part 2",
      "type":"Parameter",
      "id":4
    },
    {
      "code":"P_SCORE_A_PART_3",
      "value":0.0,
      "name":"Score A Part 3",
      "type":"Parameter",
      "id":5
    },
    {
      "code":"P_SCORE_B_PART_3",
      "value":0.0,
      "name":"Score B Part 3",
      "type":"Parameter",
      "id":6
    }
  ]
}
~~~

{:#api-calculate}
### Method: CALCULATE

**POST /api/files/$file-id/$event-id/calculate**

Производит расчет значений во вкладке "OUT" по переданым значением IN вкладки. Возвращает рассчитанные значения вкладки "OUT".
Для расчета установите необходимые значения "value" в теле ответа метода *IN-PARAMS*. Результат в виде JSON отправьте методу *CALCULATE*. В ответ Вы получите результат расчета модели в виде JSON преставления вкладки "OUT". 


<div class="well well-sm">
<b>Важно!</b> значения колонки "value" должны передаваться строками.
</div>

**Parameters**

* file-id - обязательно, ID Excel файла, по которому будет проводится событие
* event-id - обязательно, уникальный ID события

**Request body**

~~~ javascript
{
  "params" :
  [
    {
      "id" : "9", // string!
      "value" : "150.0" // string!
    },
    {
      "id" : "3",
      "value" : "0.0"
    }
  ]
}
~~~

**Responses**

* 200 - успешный расчет.
* 400 "MFP" - неправильные параметры. 
* 400 "SLE" - лимит сессий превышен.
* 404 "FNF" - файл не найден.
* 423 "CIP" - событие занято обработкой предыдущим запросом.

**Request example** <a href="#curl">*<a/>

~~~
curl -X POST --header "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" -d <<json-request-body>> http://math.engine/api/files/120/72c361803a5b116e0682581fe958fdee/calculate
~~~

**Response 200 data example**

~~~ json
{
  "status": 200,
  "data":
  [
    {
      "id" : 1,
      "market" : "Norm.Dist_300",
      "outcome" : "A",
      "coef" : 0.9259436130523682,
      "param" : 999999.0,
      "m_code" : "MATCH_NORM.DIST_300",
      "o_code" : "A",
      "param2" : 0.0,"mgp_code" : "DISTRIBUTION",
      "mn_code" : "NORM_300",
      "mgp_weight" : 1,
      "mn_weight" : 11
    },
    {
      "id" : 2,
      "market" : "Norm.Dist_300",
      "outcome" : "B",
      "coef" : "#VALUE!",
      "param" : 999999.0,
      "m_code" : "MATCH_NORM.DIST_300",
      "o_code" : "B",
      "param2" : 0.0,
      "mgp_code" : "DISTRIBUTION",
      "mn_code" : "NORM_300",
      "mgp_weight" : 1,
      "mn_weight" : 11
    }
  ]
}
~~~

### Ошибки ячеек

Любая ячейка в результате расчета может содержать строковое сообщение об ошибке. Это типичные ошибки Excel и они не относятся к ошибкам BetEngines. Поэтому код ответа будет **200 ОК**

Вероятные ошибки ячеек:

  * "#NULL!"
  * "#DIV/0!"
  * "#VALUE!"
  * "#REF!"
  * "#NAME?"
  * "#NUM!"
  * "#N/A"

Тип возвращаемого значения, будет таким же как в Excel файле. 

{:#api-release}
### Method: RELEASE

**DELETE /api/files/$file-id/$event-id**

Освобождает вычислительные ресурсы для выбранного события. 

**Parameters**

* file-id - обязательно, ID Excel файла, по которому будет проводится событие
* event-id - обязательно, уникальный ID события

**Responses**

* 204 - успешное освобождение ресурсов
* 404 "ENF" - не найденно событие с указанным event-id
* 423 "CIP" - событие занято обработкой предыдущим расчетом

**Request example** <a href="#curl">*<a/>

~~~
curl --request DELETE --header "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" http://math.engine/api/files/120/754fbcde-662c-4b3f-9f51-acb4e67913da
~~~

## Ваш сервис готов!

Теперь Вы можете проводить свои спортивные события с помощью BetEngines API и Ваших математических алгоритмов.

<hr />

<blockquote class="blockquote-reverse">
  <p>Мы много работаем над улучшением нашего сервиса, и всегда будем рады любому отзыву о его работе.</p>
  <footer>С уважением, комманда BetEngines</footer>
</blockquote>

<hr />

{:#curl}
\* Для примеров используется утилита *curl*. Вариант **curl** для своей системы можно [скачать здесь](http://curl.haxx.se/download.html).
