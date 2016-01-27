---
title: MathEngine. Integration Guide
---
## MathEngine

# Integration guide

## Introduction

This document describes the steps needed to work with BetEngines REST API. This document assumes that you have already prepared your Excel file according to the [file preparation guide](/en/doc/mengine/file-preparation) and uploaded it using Admin Web Interface ([AWI](/en/doc/user-guide)).

## BetEngines REST API usage

BetEngines usage consists of several steps:

1. [Creating an event](#event-creation) with a unique *event-id*
2. [Filling out an "IN" sheet](#in-fill-in) and sending it to the service. Gettiing values from the "OUT" sheet. Repeating this step until the event is finished.
3. [Releasing resources](#release-resources) after the event is finished

Let’s talk about each step in detail.

{:#event-creation}
### Creating an event with a unique *event-id*

For calculation speed, BE allocates computational resources for every event. 
Every event in BE uses two identificators:

* file-id
* event-id

Resources are allocated "lazily", after the first call of *CALCULATE* or *IN-PARAMS* method. So you do not need to explicitly create any event. Because of this, the first request is processed longer than subsequent ones.

Also, resources will be allocated again if the *CALCULATE* or *IN-PARAMS* method is called after *RELEASE* with the same file-id and event-id.

{:#in-fill-in}
### Filling in values on the "IN" sheet and getting result values from the "OUT" sheet

The *CALCULATE* method is dedicated to getting values

<div class="well well-sm">
<b>Note!</b> The "IN" sheet should contain two required columns: <b>"id"</b> and <b>"value"</b>, and any number of optional columns.
</div>

Requests for a single event are executed sequentially. BE will lock the event until a request returns. Calling methods on a locked event will result in a **423 "Calculation in Progress"** error.

{:#release-resources}
### Releasing resources
When interaction with BE on a particular event is finished, the client should release computation resources by calling the *RELEASE* method.

## BetEngines REST API

- [General](#api-general)
- [Authentication](#api-auth)
- [API](#api-in-params)
  * [IN-PARAMS](#api-in-params)
  * [CALCULATE](#api-calculate)
  * [RELEASE](#api-releases)


{:#api-general}
### General

Any request may return **500 errors** if an internal error occurred. If service unavailable it will return error **503 "STU"**. All responses will have Content-Type: "application/json”. Response 204 has no body as specified by HTTP protocol.

**Response structure**

~~~ json
{
  "status": 200,
  "data": {} 
}
~~~

**Error structure**

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
### Authentication

Math Engine uses [Basic Authentication](https://www.w3.org/Protocols/HTTP/1.0/spec.html#BasicAA). According to the Basic Authentication, all requests MUST contain Authorization header with authentication credentials. For example:

<pre>
GET /api/files/120/72c361803a5b116e0682581fe958fdee/in-params HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: math.engine
Accept-Language: en-us
Accept-Encoding: gzip, deflate
<b>Authorization: Basic <base64 encoded "login:password"></b>
Connection: Keep-Alive
</pre>

For login "Aladdin" and password "open sesame" base64 form of
"Aladdin:open sesame" will be "QWxhZGRpbjpvcGVuIHNlc2FtZQ==". So request will looks like this:

<pre>
GET /api/files/120/72c361803a5b116e0682581fe958fdee/in-params HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: math.engine
Accept-Language: en-us
Accept-Encoding: gzip, deflate
<b>Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==</b>
Connection: Keep-Alive
</pre>

If there is no Authorization header or credentials are invalid Math Engine will return **401 "UAR"** error with WWW-Authenticate header:

    WWW-Authenticate: Basic realm="mengine"


{:#api-in-params}
### Method: IN-PARAMS

**GET /api/files/:file-id/:event-id/in-params**

Returns IN-params for a specified file. The combination of an event-id and file-id pair should be a unique event identifier. The result of this method can be used for *CALCULATE* method.

**Parameters**

* file-id - required, ID of file to which request is applied
* event-id - required, unique event id

**Responses**

* 200 if IN-params found
* 400 "SLE" if sessions limit exceeded
* 404 "FNF" if file not found

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

**POST /api/files/:file-id/:event-id/calculate**

This method performs calculation of the *OUT* sheet based on values from the *IN* sheet. It returns calculated values from the *OUT* sheet. To perform this request, set appropriate values of the response body of IN-PARAMS method. Then send this JSON in the request body of the CALCULATE method. The result of the calculation will be an *OUT* sheet serialized to JSON.

<div class="well well-sm">
<b>Note!</b> values of the "value" column should be sent as strings.
</div>

**Parameters**

* file-id - required, ID of file to which request is applied 
* event-id - required, unique event id

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
Request body is a json object and should contain ALL file in-parameters for correct computation.

**Responses**

* 200 if calculation success
* 400 "MFP" if parameters are invalid
* 400 "SLE" if sessions limit exceeded
* 404 "FNF" if file not found
* 423 "CIP" if this event is doing calculation right now. Wait until that calculation is finishes.

**Request example** <a href="#curl">*<a/>

~~~
curl -X POST --header "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" -d <<json-request-body>> http://math.engine/api/files/120/72c361803a5b116e0682581fe958fdee/calculate
~~~

**Response 200 data example**

~~~ json
{
  "status": 200,
  "data" : 
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

### Cell errors

Any field of outcome in a response may contain an error string message if an error occured during calculation. Per cell errors are not considered as a response error, only as per cell fail, so response with cell errors will be 200 OK.

#### Possible cell errors:

  * "#NULL!"
  * "#DIV/0!"
  * "#VALUE!"
  * "#REF!"
  * "#NAME?"
  * "#NUM!"
  * "#N/A"

The outcome fields will be of type as specified in excel file.

{:#api-release}
### Method: RELEASE

**DELETE /api/files/:file-id/:event-id**

Deallocates computation resources for selected event.

**Parameters**

* file-id - required, ID of file to which request is applied
* event-id - required, unique event id

**Responses**

* 204 if deallocation was successful
* 404 "ENF" if event id is not found
* 423 "CIP" if there is a calculation with this event. Wait until it finishes.

**Request example** <a href="#curl">*<a/>

~~~
curl --request DELETE --header "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" http://math.engine/api/files/120/754fbcde-662c-4b3f-9f51-acb4e67913da
~~~

## Your service is ready!

Now you can use BetEngines API with your mathematical algorithms.
<hr />

{:#curl}
\* For request examples we are using *curl*. **curl** for your system can be [downloaded here](http://curl.haxx.se/download.html).
