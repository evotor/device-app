# API Облака для партнеров

## Список API
* [1. Изменение схемы продуктов для магазина](#001)
* [2. Получение схемы продуктов для магазина](#002)
* [3. Добавление конкретных значений экстра в товары для магазина](#003)
* [4. Получение конкретных значений экстра товаров в магазине](#004)
* [5. Получение документов чека и их экстра данных](#005)

<a name="001"></a>
### 1. Изменение схемы продуктов для магазина

Запрос:
https://test-api.evotor.ru/api/v1/inventories/stores/{store-uuid}/products/schemes
```
curl POST
    -
    H "Content-Type: application/json;charset=utf-8" -
    H "X-Auth-Token: {токен}" -
    d '[  {
        "uuid": "{uuid-схемы-продуктов-в-магазине}",
        "appId": "{uuid-приложения-в-маркетплейсе}",
        "items": [{
                "title": "Произвольный заголовок поля ввода текста",
                "editable": false,
                "regexp": "\w+", // валидный регексп
                "uuid": "{uuid-поля-1}",
                "type": "TEXT_FIELD",
                "data": {} // валидный JSON с произвольными данными доступными из JS на терминале
            },
            {
                "editable": true,
                "title": "Произвольный заголовок поля выбора из списка",
                "uuid": "{uuid-поля-2}",
                "items": [{
                        "data": {}, // валидный JSON с произвольными данными доступными из JS на терминале
                        "title": "Заголовок первого варианта выбора",
                        "value": "10"
                    },
                    {
                        "data": {}, // валидный JSON с произвольными данными доступными из JS на терминале
                        "title": "Special title",
                        "value": "20"
                    }
                ],
                "type": "DICTIONARY_FIELD"
            }
        ]
    }]
'
```
Статусы ответов:
```
200 - Успех 
400 - Ошибка "Bad Request"
401 - Ошибка "Unauthorized"
```

<a name="002"></a>
### 2. Получение схемы продуктов для магазина

Запрос:
https://test-api.evotor.ru/api/v1/inventories/stores/{store-uuid}/products/schemes
```
curl GET
    -
    H "Content-Type: application/json;charset=utf-8" -
    H "X-Authorization: {токен}"
```
Статусы ответов:
```
200 - Успех 
400 - Ошибка "Bad Request"
401 - Ошибка "Unauthorized"
```

<a name="003"></a>
### 3. Добавление конкретных значений экстра в товары для магазина

Запрос: https://test-api.evotor.ru/api/v1/inventories/stores/{store-uuid}/products/extras
```
curl - X POST - H "Content-Type: application/json" -
    H "Content-Type: application/json;charset=utf-8" -
    H "X-Auth-Token: test-token-1" - d ' [  {
        "uuid": "{uuid-экстра-поля-1}",
        "appId": "{uuid-приложения-в-маркетплейсе}",
        "key": {
            "uuid": "{uuid-продукта-связанного-с-экстра-полем}",
        },
        "scheme": { // Секция данных привязки значения экстра поля к контролу отображаемому на терминале
                    // (не требуется если экстра данные не нужно отобржатаь в меню терминал)
            "value": "20",
            "fieldId": "{{uuid-поля-из-схемы-отображения-продукта}"
        },
        "data": {}, // валидный JSON с произвольными данными доступными из JS на терминале
    }]
'
```
Статусы ответов:
```
200 - Успех 
400 - Ошибка "Bad Request"
401 - Ошибка "Unauthorized"
```

<a name="004"></a>
### 4. Получение конкретных значений экстра товаров в магазине

Запрос: https://test-api.evotor.ru/api/v1/inventories/stores/test-store/products/extras 
```
curl - XGET\ -
    H "Content-Type: application/json;charset=utf-8" -
    H "X-Authorization: {токен}"
```
Статусы ответов:
```
200 - Успех 
400 - Ошибка "Bad Request"
401 - Ошибка "Unauthorized"
```

<a name="005"></a>
### 5. Получение документов чека и их экстра данных

Запрос: https://test-api.evotor.ru/api/v1/inventories/stores/{store-uuid}/documents?deviceUuid={device_uuid}}&types=OPEN_SESSION%2CCLOSE_SESSION&ltCloseDate=2017-02-22T20%3A59%3A59.500%2B0000&gtCloseDate=2017-02-18T21%3A00%3A00.000%2B0000
```
GET
headers = {
    Content - Type = [application / json;charset = UTF - 8];
    accept - encoding = [gzip];
    x - authorization = [{
        user - application1 - token
    }]
};
req payload = ;
resp status = 200;
resp payload = [{
    "uuid": "5285ab1b-2ecb-4614-87f7-99b1486a04fe",
    "type": "OPEN_SESSION",
    "deviceId": "352398080047279",
    "deviceUuid": "8ffbe447-79d0-4fd9-823a-6da96d7f0be9",
    ...стандарнтые поля модели документа Эвотор...
    "extras": {
        "{application1-uuid}": {
            валидный_джейсон_со_значениями_эстра,
            application1 - uuid соответсвующий user - application1 - token - у
        }
    }
}]
```
Статусы ответов:
```
200 - Успех 
400 - Ошибка "Bad Request"
401 - Ошибка "Unauthorized"
```
