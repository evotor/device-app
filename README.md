# API Терминала для партнеров

## Описание API Терминала
* [Структура проекта](#001)
* [Работа с API HTTP](#002)
* [Работа с WebView](#003)
* [Работа с навигацией (API navigation)](#004)
* [Работа с БД Интеграции (API storage)](#005)
* [Работа с БД Терминала (API inventory)](#006)
* [Логирование данных (API logging)](#007)
* [Список стандартных API:](#008)
    * [1. Добавление товара в чек](#008)
    * [2. Удаление товара из чека](#009)
    * [3. Применение скидки на чек](#010)
    * [4. Получить итоговый чек при переходе к оплате](#011)
    * [5. Разделить чек по позициям на несколько чеков (группы печати)](#012)
    * [6. Получить все подключенные пинпады](#013)
    * [7. Оплатить мультичек на пинпаде](#014)
    * [8. Записать в итоговый чек экстра данные интеграции](#015)

<a name="001"></a>
### Структура проекта
Общая структура проекта:

`client.yaml` — файл с описанием разрешений для приложения, указаниями файлов, адреса view, для отображения и т.д.

Папка `client`:
* `daemons` — папка, в которой находятся файлы js, которые будут выполнятся в фоне (в примере отслеживаются события кассы);
* `uiPlugins` — папка, в которой находятся файлы js, которые выполняются перед отображением `WebView` (может не отображаться);
* `view` — папка с html файлами, стилями, скриптами и т.д., которые будут отображены в `WebView`.

Файл архива клиента распаковывается в папку `assets` андроид приложения.  
В корне должен находиться файл с описанием структуры проекта `client.yaml`.

Пример содержания:
```
version: 2
versionName: "1.0.1"
packageName: Test
appName: "testApp"
appUUID: "2e6dc4b8-fdac-48c1-8a1a-ade402863947"
iconColor: "#0f70b7"
capabilities:
  - inventory
  - storage
  - http
  - event-bus
  - receipts
daemons:
  - name: check
    events:
      - evo.receipt.opened
      - evo.receipt.productAdded
      - evo.receipt.productRemoved
      - evo.receipt.closed
      - evo.receipt.clear
      - app.suggestion.used
    behavior: check-daemon.js
plugins:
  - name: discount
    moments:
    - evo.payments.process
    - evo.payments.beforePrintReceipt
    point: before
    behavior: before-receipt-fixed.js
views:
  - name: discount-loader
    header: "Подождите"
    source: client/views/discount-loader/view.html
    scripts:
      - no-script
    styles:
      - "*.css"
  - name: launcher
    header: "Подождите"
    source: client/views/discount-loader/view.html
    scripts:
      - no-script
    styles:
      - "*.css"
```
*Где:*  
`version: 2` — Код версии приложения  
`versionName: "1.0.1"` — Версия приложения  
`packageName: Test` — Имя пакета  
`version: 2` — Код версии приложения  
`versionName: "1.0.1"` — Версия приложения  
`packageName: Test` — Имя пакета  
`appName: "testApp"` — Отображаемое пользователю имя приложения  
`appUUID: "2e6dc4b8-fdac-48c1-8a1a-ade402863947"` — UUID приложения  
`iconColor: "#0f70b7"` — Цвет иконки, если она размещена на рабочем столе    
`capabilities:`  
`  - inventory`  
`  - storage`  
`  - http`  
`  - event-bus`  
`  - receipts`  
`daemons:`  
`  - name: check` — Имя  
`    events:` — События, на которые он подписан  
`      - evo.receipt.opened`  
`      - evo.receipt.productAdded`  
`      - evo.receipt.productRemoved`  
`      - evo.receipt.closed`  
`      - evo.receipt.clear`  
`      - app.suggestion.used`  
`    behavior: check-daemon.js`  
`plugins:`  
`  - name: discount` — Имя  
`    moments:` — События, на которые он подписан  
`    - evo.payments.process`  
`    - evo.payments.beforePrintReceipt`  
`    point: before`  
`    behavior: before-receipt-fixed.js`  
`views:`  
`  - name: discount-loader`  
`    header: "Подождите"`  
`    source: client/views/discount-loader/view.html`  
`    scripts:` — список скриптов которые должны быть подключены  
`      - no-script`  
`    styles:` — список стилей которые должны быть подключены  
`      - "*.css"` — может подключить все файлы  
`  - name: launcher` — Launcher — обязательное view, если приложение на главном экране  
`    header: "Подождите"` — Заголовок, при отображении в `WebView`  
`    source: client/views/discount-loader/view.html`  
`    scripts:`  
`      - no-script`  
`    styles:`  
`      - "*.css"`  

<a name="002"></a>
### Работа с API HTTP
Для использования возможности работы с сервером посредством http запросов, в файле скрипта используется java объект, контекст которого передается в Java Script.
Работа в js с ним осуществляется, как с обычным js объектом.

Перед использованием API HTTP, необходимо явно указать необходимость использования данного функционала в скрипте:  
В начале js скрипта необходимо получить этот объект с помощью:

` var http = require('http')`

Далее в коде, после инициализации, можно вызывать метод этого объекта для отправки запроса, например:
```
function generateSuggestions(items) {
  var response = http.send({
    method : "POST",
    path : "recommendations",
    body : items
  })
  var jsonObject = JSON.parse(response)
}
```
*Где:*  
`  var response = http.send({       ` — Вызов метода  
`    method : "POST",               ` — Тип запроса  
`    path : "recommendations",      ` — Путь на сервере  
`    body : items                   ` — Тело запроса  
`  var jsonObject = JSON.parse(response)  ` — В ответе получаем строку, которую приводим к JSON объекту  
                                              (в следующей версии будет передаваться сразу объект)  

Данный функционал возможно использовать не только в сервисах-демонах, в `WebView` данный код тоже будет работать.

<a name="003"></a>
### Работа с WebView
Отображение `WebView` происходит при вызове у интерфейса `navigation` метода:
```
pushView(...)
```
куда передается адрес html страницы для открытия, где поддерживается использование css, javascript.  
Работа с Java интерфейсами внутри `WebView` несколько отличается от работы с ними в процессах-демонах:  
Доступные интерфейсы в `WebView`:

`navigation` — Работа с навигацией  
`jsData` — Интерфейс данных интеграции для передачи в webView   
`Receipt` — Работа с чеком   
`RPC` — Доступ для отправки http запросов   
`storage` — Работа с БД интеграции   
`Logger` — Работа с логированием данных   

Для работы с ними нет необходимости получать их через синтаксис required, они уже интегрированы в `WebView`.  
Работать с ними можно как с локальными переменными, без их объявления.

<a name="004"></a>
### Работа с навигацией (API navigation)
`API navigation` используется для работы с `WebView`.

Для использования функционала, необходимо явно указать о намерении использования в скрипте, работа происходит через java объект, контекст которого передается в Java Script, далее работа с ним ведется, как с обычным js объектом.
Для инициализации объекта, в начале скрипта указываем:  

Для работы с интерфейсом в сервисе-демоне:
```
var navigation = require('navigation');
```
Для работы с интерфейсом в `WebView`: ничего указывать не нужно, по умолчанию, интерфейс навигации уже передан во `WebView`, для использования обращаемся к нему, как к уже созданному объекту с именем navigation, объявлять его не нужно.  

Доступные функции в интерфейсе:
```
pushNext
pushView
```
`pushNext()` — используется для перехода к следующему экрану:  
При открытом `WebView` — закрывает его и возвращает пользователя в EvoPos, при этом в кассу передается стек операций, который представляет из себя набор действий: добавление товара в чек, удаление товара из чека, применение скидки к чеку или к отдельно выбранному товару.

`pushView(String viewLocation, String data)` — где:
* `viewLocation` — адрес html страницы для открытия в `WebView`
* `data` — строка для данных, которые должны быть переданы в `WebView`, обычно используется json формат  

Пример работы:
```
navigation.pushView("client/views/suggestion-list/view.html", {
  suggestions: suggestedProducts,
  receipt: receipt
});
```
Для получения данных в открывшемся `WebView` используется интерфейс jsData, имеющий метод `getData()`, возвращающий данные в строковом представлении, переданные через метод `pushView()`.  

Пример использования:
```
var passedData  = JSON.parse(jsData.getData());
```

<a name="005"></a>
### Работа с БД Интеграции (API storage)

`storage` – система хранения данных в формате ключ – значение.

Для работы с API необходимо в манифесте приложения указать:
```
capabilities:
 - storage
```

Объект для работы с API вызывается функцией:
```
var storage = require('storage')
```

Далее используются две функции:

* Сохранение данных:
```
storage.set(key, value)
```
Функция возвращает `true` в случае успешного сохранения, `false` если произошла ошибка.
key и value – строковые переменные

* Получение данных:
```
storage.get(key)
```
Функция возвращает строковую переменную ранее записанную в хранилище, либо `null`, если значение не было найдено.
`key` – строковая переменная

<a name="006"></a>
### Работа с БД Терминала (API inventory)

`API inventory` используется для доступа к базе данных устройства, конкретнее, к таблице, содержащей информацию о товарах.
Для использования функционала, необходимо явно указать о намерении использования в скрипте.
Работа происходит через java объект, контекст которого передается в Java Script, далее работа с ним ведется, как с обычным js объектом.
Для инициализации объекта, в начале скрипта указываем:
```
var inventory = require('inventory');
```
После этого, мы имеем доступ к методам этого объекта, на данный момент, реализован метод, для получения информации по конкретному товару в базе данных устройства.
Для ее получения, необходимо передать в функцию уникальный uuid товара.
Например:
```
function getProduct(productUID){
        return inventory.getProduct(productUID);
    }
```
Результатом работы функции будет JSON объект в строковом представлении, вида:
```
{
  "ID":"136",
  "UUID":"1196da34-e4a8-4915-8e92-bd7792875d76",
  "CODE":"4",
  "CODE_UPPER_CASE":"4",
  "NAME":"вино апсны",
  "NAME_UPPER_CASE":"ВИНО АПСНЫ",
  "IS_GROUP":"0",
  "IS_FAVORITE":"0",
  "MEASURE_ID":"1",
  "PRICE_OUT":"20000",
  "COST_PRICE":"0",
  "QUANTITY":"-1000",
  "TAX_NUMBER":"VAT_18",
  "ABBREVIATION":"ВНП",
  "TILE_COLOR":"-26624",
  "TYPE":"NORMAL",
  "ALCOHOL_BY_VOLUME":"0",
  "ALCOHOL_PRODUCT_KIND_CODE":"0",
  "TARE_VOLUME":"0",
  "SELL_FORBIDDEN":"0",
  "DESCRIPTION":"",
  "ARTICLE_NUMBER":"",
  "ARTICLE_NUMBER_UPPER_CASE":""
}
```
Если, товар с указанным uuid не будет найден в базе, то результатом будет строка с пустым JSON объектом:
```
{
}
```
Данный функционал возможно использовать не только в сервисах-демонах, в `WebView` данный код тоже будет работать.

<a name="007"></a>
### Логирование данных (API logging)

Функционал для логгирования
Объект, через который осуществляется логгирование получается функцией `require`:
```
var logger = require('logger')
```
Далее у объекта вызывается функция:
```
logger.log(value)
```
После выполнения функции в logcat устройства будет выведена строка `value`

<a name="008"></a>
### 1. Добавление товара в чек
Управление чеком доступно через `receipt api`, для этого используем метод:  
`receipt.addPosition(uuid: String)` — добавление товара в чек

<a name="009"></a>
### 2. Удаление товара из чека
Управление чеком доступно через `receipt api`, для этого используем метод:  
`receipt.removePosition(uuid: String)` — удаление из чека товара, который уже был добавлен через 'addPosition'

<a name="010"></a>
### 3. Применение скидки на чек
Управление чеком доступно через `receipt api`, для этого используем метод:  
`receipt.applyReceiptDiscountPercent(discount: Double)` — применение скидки ко всему чеку, процентное значение

<a name="011"></a>
### 4. Получить итоговый чек при переходе к оплате

Для получения всего чека используем:

`receipt.getReceipt()`
Возвращает строку в JSON формате:
```
 {
    "receiptData": {
        "totalSum": "218.50",
        "discountPercents": "0.000000",
        "totalSumWithoutDiscount": "218.50",
        "positionDiscountSum": "0.00",
        "positionsCount": "4",
        "extraData": "{}"
    },
    "receiptPositions": [{
        "uuid": "070efd1b-4f53-401a-b5c1-cb31b5e9072d",
        "type": "NORMAL",
        "code": "1",
        "measure": "шт",
        "price": "56",
        "priceWithDiscount": "56",
        "quantity": "1"
    }, {
        "uuid": "9d2fdacd-969c-41f2-b360-a5ebbddcbe9e",
        "type": "NORMAL",
        "code": "131",
        "measure": "шт",
        "price": "30.5",
        "priceWithDiscount": "30.5",
        "quantity": "5"
    }, {
        "uuid": "7c916c19-756d-4b3d-806e-63c090080a3d",
        "type": "NORMAL",
        "code": "129",
        "measure": "шт",
        "price": "9",
        "priceWithDiscount": "9",
        "quantity": "1"
    }, {
        "uuid": "9dd36300-647e-4863-81b2-eb9591bc599b",
        "type": "NORMAL",
        "code": "127",
        "measure": "шт",
        "price": "1",
        "priceWithDiscount": "1",
        "quantity": "1"
    }]
}
```

<a name="012"></a>
### 5. Разделить чек по позициям на несколько чеков (группы печати)

Группы печати, где для задания группы печати на позицию, используется метод:
```
edited
receipt.addPositionPrintGroup(JSON String)
 в качестве параметра
 var addPositionPrintGroup = {
    "uuid": "e82a113b-0d76-424f-9ad5-c6595ca57770", 
    "code": "4", — код товара
    "printGroupId" : "4dc2-3fcd", 
    "printGroupIsFiscal" : true, 
    "printGroupOrgName" : "Andrew", 
    "printGroupOrgInn": "88005553535", 
    "printGroupPaymentSum" : "123.00" 
};
edited
```
*Где:*   
``"uuid": "e82a113b-0d76-424f-9ad5-c6595ca57770"`` — uuid товара  
``"code": "4"`` — код товара  
``"printGroupId" : "4dc2-3fcd"`` — id группы печати  
``"printGroupIsFiscal" : true`` — признак фискальности  
``"printGroupOrgName" : "Andrew"`` — имя группы  
``"printGroupOrgInn": "88005553535"`` — ИНН  
``"printGroupPaymentSum" : "123.00"``— Сумма товаров в группе  

<a name="013"></a>
### 6. Получить все подключенные пинпады
**В процессе разработки**

<a name="014"></a>
### 7. Оплатить мультичек на пинпаде
```
receipt.addPaymentOperation(JSON String)
```

Принимает на вход принимает JSON, который описывает структуру оплаты для разбиения общего чека на платежи:
```
var paymentOperation = {
  "uuid": "1ef45g5",
  "deviceUUID": "smth",
  "paymentSum" : "123.00",
  "isCashless" : true,
  "printGroups" : [
  {
      "printGroupId" : "4dc2-3fcd",
      "printGroupIsFiscal" : true,
      "printGroupOrgName" : "Andrew",
      "printGroupPaymentSum" : "123.00",
      "printGroupOrgInn": "88005553535"
   }
  ]
};
```
*Где:*    
``"uuid": "1ef45g5"`` — юид оплаты  
``"deviceUUID": "smth"`` — юид устройства для оплаты  
``"paymentSum" : "123.00"`` — сумма платежа  
``"isCashless" : true`` — признак картой/нал  
``"printGroups" :``  — список групп печати в оплате  


<a name="015"></a>
### 8. Записать в итоговый чек экстра данные интеграции
Все необходимые данные интеграция может добавить в нужном ей формате как дополнительную информацию по документу чека продажи:
```
receipt.addExtraReceiptData(String extraData)
```
