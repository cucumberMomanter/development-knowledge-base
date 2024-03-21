#openAPI #manifestFirst
## Ссылки
[Проектирование OpenAPI спецификации](https://habr.com/ru/articles/705698/)
[Руководство по OpenAPI](https://starkovden.github.io/openapi-tutorial-overview.html)
## Helpful information
- При построении API, request и response лучше делать объектами сразу, даже если нам нужно передавать одно поле. Это необходимо делать для обеспечения обратной совместимости с нашим API. Т.к. мы всегда будем принимать/отдавать один и тот же объект, а вот сам объект мы сможем расширять уже как захотим в дальнейшем с минимальными правками.
  - Пример курильщика
    ```yaml
    /path/birthday:
      post:
        summary: пример endpoint с плохой обратной совместимостью
        operatinId: getBirthday
        requestBody:
          content:
            application/json:
              schema:
                type: string
                format: date
        responses:
          200:
            content:
              application/json:
                schema:
                    type: string
                    format: date
    ```
  - Хороший пример
    ```yaml
    /path/birthday:
      post:
        summary: пример endpoint с хорошей обратной совместимостью
        operatinId: getBirthday
        requestBody:
          content:
            application/json:
              schema:
                birthdayData:
                  type: object
                  properties:
                    birthday:
                      type: string
                      format: date
        responses:
          200:
            content:
              application/json:
                schema:
                  birthdayData:
                    type: object
                      properties:
                        birthday:
                          type: string
                          format: date
    ```
- 2

## Manifest first
> Манифест - соглашение по взаимодействию.
> В Open API манифест - это спецификация.

Подход <font style="color:red">Manifest first</font> - первым создается манифест (спецификация). Он является источником истины для app. Относительно него реализуется уже функционал frontend и backend.

Основная идея в том, что все разночтения по использованию API должны, по возможности, решаться на уровне спецификации. После утверждения спецификации можно начинать разрабатывать параллельно frontend и backend, что увеличивает скорость разработки app в целом.
А, например, через Postman (или другие сервисы) можно <font style="color:violet">имитировать запросы</font> от другой системы, пока она разрабатывается, <font style="color:violet">на основе спецификации</font>.

## Состав спецификации
- св-во <font style="color:green">openapi</font> - версия используемой спецификации.
- секция <font style="color:green">info</font> - описание данного API (название, версия и пр.)
- секция <font style="color:green">servers</font> - куда идут обращения (начальные пути URL серверов API)
- секция <font style="color:green">paths</font> - точки обращения (endpoints) API, к которым идет обращение (содержат в себе запрос и возможные ответы).
- секция <font style="color:green">components</font> - схемы и параметры, совместно используемые в endpoint'ах.
- и прочее.

### Пример спецификации
```yaml
openapi: 3.1.0  
info:  
  version: 1.0.0  
  title: Rest Api для сервиса Calendar  
servers:  
  - url: http://localhost:8891/api/v2  
paths:  
  /info/calendar/session:  
    post:
      operationId: getSession  
      parameters:  
        - in: query  
          name: date  
          required: false  
          schema:  
            type: string  
            format: date-time  
      requestBody:  
        required: true  
        content:  
          application/json:  
            schema:  
              $ref: "../common/trade/instrument.yaml#/components/schemas/MarketInstrumentKey"  
      responses:  
        200:  
          description: "desc"  
          content:  
            application/json:  
              schema:  
                $ref: "#/components/schemas/Session"  
  /info/calendar/schedule:  
    post:  
      parameters:  
        - in: query  
          required: true  
          name: startDate  
          schema:  
            type: string  
            format: date  
        - in: query  
          required: true  
          name: endDate  
          schema:  
            type: string  
            format: date  
      requestBody:  
        required: true  
        content:  
          application/json:  
            schema:  
              $ref: "../common/trade/instrument.yaml#/components/schemas/MarketInstrumentKey"  
      operationId: getSchedule  
      responses:  
        200:  
          description: "desc"  
          content:  
            application/json:  
              schema:  
                type: array  
                items:  
                  $ref: "#/components/schemas/Interval"  
components:  
  schemas:  
    Interval:  
      type: object  
      properties:  
        open:  
          type: string  
          format: date-time  
        close:  
          type: string  
          format: date-time  
    Day:  
      type: object  
      properties:  
        date:  
          type: string  
          format: date  
        isWorkingDay:  
          type: boolean  
        nextWorkingDay:  
          type: string  
          format: date  
        previousWorkingDay:  
          type: string  
          format: date  
    Session:  
      type: object  
      properties:  
        date:  
          type: string  
          format: date-time  
        openCloseTime:  
          type: array  
          items:  
            $ref: "#/components/schemas/Interval"  
        nextOpenCloseTime:  
          type: array  
          items:  
            $ref: "#/components/schemas/Interval"  
        currentOrLastOpenCloseTime:  
          type: array  
          items:  
            $ref: "#/components/schemas/Interval"
```
### Секция серверов (servers)
Указывает перечень <font style="color:violet">начальных URL</font> для обращения к API. Обычно это — схема и домен сервера. 

```yaml
servers:  
  - url: http://localhost:8891/api/v2
    description: Dev server
  - url: https://api-links.alpha.ru
  - url: http://localhost:3333
    description: Local server
```
По умолчанию клиенты используют первый указанный сервер.
### Секция путей (paths)
> path - это endpoint

Описывает точку обращения. Внутри пути может быть определено несколько API-методов, разделяемых благодаря указанию HTTP-метода.
```yaml
paths:  
  /info/calendar/session:  
    post:
      summary: Получение сессии по инструменту на дату
      operationId: getSession  
      parameters:  
        - in: query  
          name: date  
          required: false  
          schema:  
            type: string  
            format: date-time  
      requestBody:  
        required: true  
        content:  
          application/json:  
            schema:  
              $ref: "../common/trade/instrument.yaml#/components/schemas/MarketInstrumentKey"  
      responses:  
        200:  
          description: "desc"  
          content:  
            application/json:  
              schema:  
                $ref: "#/components/schemas/Session" 
```

- `/info/calendar/session` - URL обращения (относительно выбранного сервера)
- `post` - <font style="color:violet">объект operaions</font> - это HTTP методы обращения: GET, POST, PUT, PATCH, DELETE.
  - `tags` - групповое имя для организации путей в интерфейсе Swagger. Swagger UI сгруппирует конечные точки под заголовками тегов.
  - `summary` - краткое описание пути. Ограничивают описание только 5-10 словами.
  - `description` - Полное описание пути. Может содержать неограниченное количество деталей.
  - `externalDocs` <font style="color:violet">(объект)</font> - ссылка на документацию с доп. информацией о пути.
  - `operationId` - уникальный идентификатор пути. (<font style="color:violet">имя метода endpoint'а</font>)
    Формируется из глагола действия (add — добавить, update — обновить и т.д.) + название сущности (Project, User, Autobuyer, Text и т.п.) + опционально вспомогательная информация.
  - `parameters` <font style="color:violet">(объект)</font> -  Параметры, принимаемые путем. Не включает параметры тела запроса (`requestBody`). Может ссылаться на объект из `components`.
    - `name` - имя параметра
    - `in` - место параметра. Возможные значения:
      -  `query` - параметр будет передан в строке запроса URL (<font style="color:#00CED1;">?param=value</font>)
      - `path` - параметр будет передан как часть URL-пути (<font style="color:#00CED1;">/resource/{param}</font>)
      - `header` - параметр будет передан в заголовке запроса (<font style="color:#00CED1;">HeaderName: value</font>)
      - `cookie` - параметр будет передан как куки в запросе (<font style="color:#00CED1;">Cookie: param=value</font>)
    - `description` - описание параметра.
    - `required` - обязательность заполнения параметра.
    - `deprecated` - является ли параметр устаревшим.
    - `allowEmptyValue` - позволяет ли параметр передавать пустое значение.
    - `style` - как данные параметра сериализуются.
    - `explode` - расширенный параметр, связанный с массивами
    - `allowReserved` - разрешены ли зарезервированные символы.
    - `schema` <font style="color:violet">(объект)</font> - Схема или модель для параметра. Схема определяет структуру входных или выходных данных. Обратите внимание, что `schema` также может содержать объект `example`.
    - `example` - пример типа носителя. Если объект `example` содержит примеры, эти примеры появляются в Swagger UI, а не в содержимом объекта `example`.
    - `examples` <font style="color:violet">(объект)</font> -  Пример типа носителя, включающий схему.
  - `requestBody` <font style="color:violet">(объект)</font> -  детали параметра тела запроса для этого пути. Может ссылаться на объект из `components`.
  - `response` <font style="color:violet">(объект)</font> - ответы, предоставленные на запросы по этому пути. Ответы используют стандартные коды состояния. Может ссылаться на объект из `components`.
  - `callbacks` <font style="color:violet">(объект)</font> - 
  - `deprecated` <font style="color:violet">(boolean)</font> - Является ли путь устаревшим. Можно опустить, если вы не хотите указать устаревшее поле.
  - `security` <font style="color:violet">(объект)</font> - Метод безопасной авторизации, используемый с операцией. Этот объект добавляется на уровне пути, только если нужно перезаписать объект `security` на корневом уровне. Имя определяется объектом `securitySchemes` в объекте `components`
  - `servers` <font style="color:violet">(объект)</font> -  Объект servers, который может отличаться от глобального объекта servers для этого пути.
### Секция компонентов (components)
> В `components` хранятся переиспользуемые определения, которые могут появляться в нескольких местах в документе спецификации.

В `components` можно описывать переиспользуемые структуры для секций:
- `schemas` схемы
- `responses` ответы
- `parameters` параметры запросов
- `examples` примеры
- `requestBody` тело запросов
- `headers` заголовки
- `securitySchemes` 
- `links` ссылки
- `callbacks` колбеки
```yaml
components:  
  schemas:  
    Interval:  
      type: object  
      properties:  
        open:  
          type: string  
          format: date-time  
        close:  
          type: string  
          format: date-time
    Session:  
      type: object  
      properties:  
        date:  
          type: string  
          format: date-time  
        openCloseTime:  
          type: array  
          items:  
            $ref: "#/components/schemas/Interval"  
        nextOpenCloseTime:  
          type: array  
          items:  
            $ref: "#/components/schemas/Interval"  
        currentOrLastOpenCloseTime:  
          type: array  
          items:  
            $ref: "#/components/schemas/Interval"
```
При создании спецификации нужно стараться переиспользовать параметры, используя возможность ссылок в OpenAPI — `$ref`.
`$ref` ссылается на секцию компонентов.

### Objects
#### Schema Object
> Schemas в OpenAPI - это описание структуры данных (например, объектов, массивов, строк и т. д.).
> Использование схем в OpenAPI обеспечивает однородность и консистентность структуры данных в API.

##### Структура
- `type` - определяет тип данных
  - Основные типы данных, поддерживаемые в стандарте OpenAPI:
    - `string`
    - `number` - целые и дробные числа
    - `integer` - целое число
    - `boolean`
    - `array`
      - `items` - описывает элементы массива
      - `minItems` и `maxItems` - устанавливают минимальное и максимальное количество элементов в массиве
    - `object`
      - `properties` - содержит описание свойств(полей) объекта
    - `null`
  - Расширенные типы данных:
    -  `date`
    - `date-time`
    - `password`
    -  `byte`
    - `binary`
    -  и пр.
- `title` и `description` - описание схемы
- `format` - определяет формат данных. На основе данного св-ва генераторы кода могут подбирать соответствующие типы данных.
  - `date` - формат для даты (например, "2019-12-31")
  - `date-time` - формат для даты и времени (например, "2019-12-31T23:59:59Z")
  - `password` - формат для пароля (например, "")
  - `byte` - формат для данных в виде последовательности байтов (например, base64-encoded image)
  - `binary` - формат для бинарных данных (например, файлы)
  - `email` - формат для электронной почты (например, "example@example.com")
  - `hostname` - формат для хостнейма (например, "example.com")
  - `ipv4` - формат для IPv4 адреса (например, "192.168.1.1")
  - `ipv6` - формат для IPv6 адреса (например, "2001:0db8:85a3:0000:0000:8a2e:0370:7334")
  - `uri` - формат для URI (Uniform Resource Identifier) (например, "https://www.example.com")
  - и пр.
  - 
- `example` - пример данных, соответствующих структуре
- `nullable` (boolean) - указание, что <font style="color:violet">объект</font> может содержать значение null
- `default` - устанавливает значение по умолчанию
- `minimum` и `maximum` - устанавливают минимальное и максимальное значения <font style="color:violet">для числовых данных</font>
- `minLength` и `maxLength` - минимальная и максимальная длина значения <font style="color:violet">для строк</font>
- `required` - указывает на обязательные свойства объекта
- `additionalProperties` - определяет, могут ли содержаться дополнительные свойства, которые не описаны в схеме
- `enum` - список допустимых значений <font style="color:violet">для строк</font>
- `pattern` - регулярное выражение, которому должно соответствовать значение (<font style="color:violet">для строк</font>)

##### Примеры
###### Общий пример
```yaml
type: object
title: Example Schema
description: This is an example schema
properties:
  stringProperty:
    type: string
    format: email
    description: Email address
    minLength: 1
    maxLength: 2
    default: example@example.com
  numberProperty:
    type: number
    minimum: 1
    maximum: 10
  booleanProperty:
    type: boolean
    default: false
  arrayProperty:
    type: array
    items:
      type: string
    minItems: 1
    maxItems: 5
  objectProperty:
    description: desc objectProperty
    type: object
    properties:
      nestedProperty:
        type: string
        enum: [active, inactive]
    nullable: true
required:
  - stringProperty
additionalProperties: false
```
###### Примеры со string объектами
```yaml
stringObjects:
#schema тут можно не писать, т.к. при ссылке на именованный объект схема там и так будет
  schema:
    type: object
    properties:
      birthDate:
        description: Дата рождения
        type: string
        pattern: "YYYY-MM-DD"
        example: "1996-02-27"
      currency:
        type: string
        enum: [RUB, USD]
        example: RUB
      email:
        description: Email address
        type: string
        format: email
        default: example@example.com
        minLength: 1
        maxLength: 35
```
