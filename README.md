# QR
## Тестовый Smoker сервер
[Смокер](https://smocker.dev/) имитирует работу настоящего бэкенда для клиентских модулей. Возвращает статические JSON ответы по конкретным запросам.
## Структура
```
.
├── compose.yaml
├── data
│   ├── creds
│   ├── encrypted
│   ├── keys
│   ├── payloads
│   ├── requests
│   ├── signs
│   └── views
├── mocks
│   ├── accept_enatance_for_user1_with_new_private_key.yaml
│   ├── accept_enterance_for_user_1.yaml
│   ├── reject_enterance_for_user_2.yaml
│   └── set_new_private_ley_while_eneterance_for_user_1.yaml
└── README.md
```

## Запуск сервера
В [`.env`](.env.example) указываются
- `SMOCKER_HTTP_PORT` - порт сконфигурированных эндпоинтов
- `SMOCKER_ADMIN_PORT` - порт панели администратора и эндпоинта для загрузки конфигуарация

Для запуска смокера необхоимо ввести
```
docker compose up -d
```

Для остановки использовать
```
docker compose down
```

При выполнение данной команды запускаются два контейнера: сам смокер (`qr-smocker-1`) и временный контейнер (`qr-load-mocks-1`), в который монтируется директория с конфигурациями моковых эндпоинтов.

## Контракт взаимодействия
Для доступа модуля к серверу предоставляется только один эндпоинт: `http(s)://<host>:<port>/enter` с методом `POST`

### Тело запроса от модуля:
#### Объект (1)

    {
        "payload": 
        {
            "uuid": "c2e0ccc1-7028-491e-bed0-c3cbd5aa93a6",
            "totp": "251720"
        },
        "signature": "Q8JQYs6pmpwA....U2xypzed"
    }

### Тело ответа от сервера:
#### Объект (2)

    {
        "result": "ACCEPTED"/"REJECTED"
    }

**или**

#### Объект (3)

    {
        "private_key": "LS0tLS1CR....gS0VZLS0tLS0K" // в кодировке base64
    }

## Данные
В качестве данных для моковых запросов/ответов используются

### [data/creds](/data/creds/)
- Идентификаторы пользователей `UUIDv4` \
    https://www.uuidgenerator.net/version4
- Одноразовый пароль `TOTP` \
    https://totp.danhersam.com/ \
    SECRET_KEY: JBSWY3DPEHPK3PXP

### [data/keys](/data/keys/)
Приватные и публичные ключи для асимметричного шифрования и цифровой подписи, сгенерированные через openssl
- Алгоримт `RSA`
- Длина приватного ключа `3072` бит

### [data/payloads](/data/payloads/)
Композиция `UUID` и `TOTP` в формате `JSON`

### [data/signs](/data/signs/)
Цифровые подпсии для `payloads` с использованием приватных ключей и хэш-функции `sha256`
|Подпись|Использлванный приватный ключ|
|---|---|
|payload1.sig|private1.pem|
|payload2.sig|pricate1.pem|
|payload2_new_pricate.sig|private2.pem|
|payload3.sig|private1.pem|

### [data/views](/data/views/)
Представление цифровых подписей запросов и ***нового*** приватного ключа в кодировке `base64`

### [data/requests](/data/requests/)
Копмозиция `payloads` с цифровыми подписями, именно эти запросы ожидает смокер    

## Конфигурации
Синтаксис задания конфигураций описан в [документации](https://smocker.dev/docs/technical-documentation/mock-definition) \
В данном проекте настроено 4 кофигурации:

|Конфигурация|Описание|Ожидаемый запрос|Ожидаемый ответ|
|---|---|---|---|
|[`mocks/accept_enterance_for_user_1`](/mocks/accept_enterance_for_user_1.yaml)|Разрешить доступ для пользовател №1 с подписью ***основным*** ключом|[Объект (1)](#объект-1)|[Объект (2)](#объект-2)|
|[`mocks/reject_enterance_for_user_2`](/mocks/reject_enterance_for_user_2.yaml)|Запретиить доступ для пользователя №2 с подписью ***основным*** ключом|[Объект (1)](#объект-1)|[Объект (2)](#объект-2)|
|[`mocks/set_new_private_ley_while_eneterance_for_user_1`](/mocks/set_new_private_ley_while_eneterance_for_user_1.yaml)|Требование модулю установить ***новый*** приватный ключ|[Объект (1)](#объект-1)|[Объект (3)](#объект-3)|
|[`accept_enatance_for_user1_with_new_private_key`](/mocks/accept_enatance_for_user1_with_new_private_key.yaml)|Разрешить доступ для пользователя №1 с подписью ***новым*** ключом|[Объект (1)](#объект-1)|[Объект (2)](#объект-2)|