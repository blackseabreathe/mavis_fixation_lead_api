# Мавис, API фиксации лидов
Документация описывает возможные ответы от API при попытке зарегистрировать лид. Ответы охватывают случаи ошибок валидации, проверку уникальности номера телефона и статус фиксации лида.

## Общая структура ответа
```json
{
  "result": [boolean],
  "error": [int],         // Присутствует только при ошибке
  "message": [string],    // Сообщение об ошибке
  "fixation_deadline": [string|null] // Присутствует при фиксации лида
}
```

## 🟡 Токен должен быть передан в заголовках запроса в параметре X-AGENT-TOKEN

Запрос должен быть отправлен методом `GET`. 


### ❌ Ошибки в запросе (400, 401, 405)

#### 1. Отправлен запрос методом, отличным от `GET`
```
{
  "result": false,
  "error": 405,
  "message": "Request method is not supported"
}
```

#### 2. Не передан токен
```
{
  "result": false,
  "error": 401,
  "message": "Token is missing"
}
```

#### 3. Некорректный токен
```
{
  "result": false,
  "error": 400,
  "message": "Token is incorrect"
}
```

#### 4. Тело запроса пустое
```
{
  "result": false,
  "error": 400,
  "message": "No data was transferred"
}
```




### ❌ Ошибки валидации (400)

#### 1. Отсутствует имя клиента
```
{
  "result": false,
  "error": 400,
  "message": "Client name is too short"
}
```

#### 2. Отсутствует фамилия клиента:
```
{
  "result": false,
  "error": 400,
  "message": "Client surname is too short"
}
```

#### 3. Отсутствует телефон клиента или передано менее 11 цифр:
```
{
  "result": false,
  "error": 400,
  "message": "Phone must consist of 11 digits"
}
```

#### 4. Отсутствует имя агента:
```
{
  "result": false,
  "error": 400,
  "message": "Agent name is too short"
}
```

#### 5. Отсутствует телефон агента или передано менее 11 цифр:
```
{
  "result": false,
  "error": 400,
  "message": "Agent phone must consist of 11 digits"
}
```

#### 6. Отсутствует название агентства:
```
{
  "result": false,
  "error": 400,
  "message": "Agency name is too short"
}
```



### ⚠️ Статусы фиксации (200, 208, 406)

#### 1. Телефон неуникальный и лид НЕ на стадии Фиксация:
```
{
  "result": false,
  "error": 406,
  "message": "not unique"
}
```

#### 2. Телефон неуникальный, лид на стадии Фиксация:
```
{
  "result": false,
  "error": 208,
  "message": "Lead was already fixed",
  "fixation_deadline": "2025-07-03 20:00:00"
}
```


### 3. Телефон уникален, успешная фиксация:
```
{
  "result": true,
  "message": "unique",
  "fixation_deadline": "2025-07-03 20:00:00"
}
```

Во всех ответах HTTP статус идентичен параметру `error`, кроме успешной фиксации. Например, не отправили номер телефона агента, в ответе получили `{... "error": 400, ...}`, значит и HTTP статус тоже 400.

При успешной фиксации 
```diff 
+ HTTP статус - 200
```
параметр `error` не возвращается.

В параметре `message` возвращается полное описание ошибки


# Curl пример запроса
```diff
- Внимание! Есть обязательные параметры в запросе
```

```
$endpoint = 'https://crm02.r2d.ru/local/samoliot-app/lead-fixation/fixation/v2/';

$params = [
    "phone" => "+7 (912) 000-00-11", // Номер телефона клиента, любой формат номера телефона, но состоящий из 11 цифр (обязательный)
    "surname" => "Безфамильный", // Фамилия клиента (обязательный)
    "name" => "Безимян", // Имя клиента (обязательный)
    "middleName" => "Безотчествович", // Отчество клиента
    "agency" => "Ми-6", // Название агентства (обязательный)
    "agent" => [
        "name" => "Агент Джеймс Бонд 007", // Имя агента (обязательный)
        "phone" => "7 900 232 52-23", // Телефон агента, любой формат номера телефона, но состоящий из 11 цифр (обязательный)
    ],
];

// Заголовки
$headers = [
    'Accept: application/json', // формат JSON
    'Content-Type: application/json', // формат JSON
    'X-AGENT-TOKEN: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx'
];


$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_URL, $endpoint.'?'.http_build_query($params));
curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);

$result = json_decode(curl_exec($curl), true);
if (curl_errno($curl)) {
    echo 'Ошибка cURL: ' . curl_error($curl);
}

$http_code = curl_getinfo($curl, CURLINFO_HTTP_CODE); // получим HTTP статус
curl_close($curl);

echo "http code: $http_code<br><br><br>";

echo '<pre>';
print_r($result);
```
