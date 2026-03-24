# Гайд на API MAX

### Пример можно посмотреть [здесь](https://github.com/PronikFire/Client-Max-Api)
### Нашли ошибку или хотите задать вопрос? Создайте *Issue*

## Общая информация

У Max существует два API:

* **WSS (WebSocket)** - для web-версии
* **TLS** - для приложений

По факту это один и тот же API, но разница между ними все же есть. Как пример, в web-версии вырезали аутентификацию по номеру телефону

---

## Web API

Наиболее простой API с понятной структурой и самым легким способом анализа.

### Как анализировать

Анализ можно проводить прямо в браузере с помощью панели разработчика:

1. Заходим на сайт: [web.max.ru](https://web.max.ru)
2. Открываем панель разработчика:
   * `Ctrl + Shift + J`, либо
   * ПКМ -> **Проверить** (Текст зависит от браузера)
3. Переходим во вкладку **Network**
4. (Опционально) В фильтрах сверху выбираем **Socket**
5. Перезагружаем страницу
6. Находим и открываем WebSocket-подключение:
   * если выполнен пункт 4 - оно будет первым и единственным
7. Переходим во вкладку **Messages**

### Содержание сообщений

Пример:

> {"ver":11,"cmd":0,"seq":0,"opcode":6,"payload":{*здесь payload*}}

| Поля | Описание |
| - | - |
| ver | Версия протокола |
| cmd | Команда (см. [здесь](#значения-cmd)) |
| seq | Номер операции |
| opcode | Код операции (см. [здесь](#список-opcode)) |
| payload | Полезная нагрузка - содержание сообщения |

Сообщения кодируются в JSON

---

## App API

Здесь начинается самое интересное. В отличие от web-версии, в приложении нет простого пути анализа, а структура пакетов другая

### Как анализировать

Здесь нам не обойтись без стороннего софта

1. Скачиваем Windows-версию приложения:
   * [https://download.max.ru/#desktop](https://download.max.ru/#desktop)
2. Устанавливаем
   * Я использовал виртуальную машину, но это не принципиально
3. Скачиваем и устанавливаем **mitmproxy**:
   * [https://www.mitmproxy.org](https://www.mitmproxy.org)
4. Запускаем **mitmweb** (веб-интерфейс - более удобен)
5. Устанавливаем сертификата
   * ``` C:\Users\%User%\.mitmproxy\mitmproxy-ca-cert.cer ``` \
    Устанавливать для: **Локального компьютера** \
    Хранилище: **Доверенные корневые центры сертификации**

### Настройка mitmproxy

В интерфейсе mitmproxy:

1. Переходим во вкладку **Capture**
2. Включаем **Local Applications**
3. Выбираем `max.exe`
   * Max должен быть запущен на фоне
5. Переходим во вкладку **Flow List**
6. Очищаем список:
   * **File -> Clear All**
7. (Опционально) Перезапускаем Max.

Готово.

### Советы по анализу сообщений

В mitmproxy после выбора соединения в правом верхнем углу можно выбрать режим **View**.

Наиболее полезные режимы:

* **Hex Dump**
* **Hex Stream** - то же самое, но только в одну строку

Для анализа **Hex Stream** удобно использовать:

* [hexed.it](https://hexed.it) - отличный инструмент для работы с бинарными данными.
* [HxD](https://mh-nexus.de/en/hxd/) - офлайн аналог

Также советую использовать jadx на мобильной версии. Это даст больше понимания

---

### Содержание сообщений

Заголовок - 10 байт.

| Номер байта | Описание |
| - | - |
| 1 | ver - версия протокола |
| 2 | cmd - команда (см. [здесь](#значения-cmd))  |
| 3..4 | seq - порядковый номер операции |
| 5..6 | opcode - код операции (см. [здесь](#список-opcode)) |
| 7 | cof - степень сжатия LZ4 (0 — сжатие не используется) |
| 8..10 | Длина payload в байтах |
| 11.. | Payload - содержание пакета (Кодируется в формате **MsgPack**) |

---

## Значения `cmd`

> [!NOTE]
> Актуально и для WebSocket

| Значение | Описание                                     |
| -------- | -------------------------------------------- |
| 0        | Запрос                                       |
| 1        | Ответ                                        |
| 2        | ??? (видел, но не уверен, какую роль играет) |
| 3        | Ошибка                                       |

## Сжатие

Байт *cof* - это коэффициенту сжатия.
> [!NOTE]
> (исходная длина / длина при сжатии) + 1
Округлёный до целых.

Сжатие используется только когда длина payload больше 32.

---

## Список `opcode`

> [!IMPORTANT]
> Большая часть была собраны из Android приложения, версии 25.21.3. Версия протокола: 11.

| Opcode | Описание |
| - | - |
| 1 | PING |
| 2 | DEBUG |
| 3 | RECONNECT |
| 5 | LOG |
| 6 | SESSION_INIT |
| 16 | PROFILE |
| 17 | AUTH_REQUEST |
| 18 | AUTH |
| 19 | LOGIN |
| 20 | LOGOUT |
| 21 | SYNC |
| 22 | CONFIG |
| 23 | AUTH_CONFIRM |
| 25 | PRESET_AVATARS |
| 26 | ASSETS_GET |
| 27 | ASSETS_UPDATE |
| 28 | ASSETS_GET_BY_IDS |
| 29 | ASSETS_ADD |
| 31 | SEARCH_FEEDBACK |
| 32 | CONTACT_INFO |
| 33 | CONTACT_ADD |
| 34 | CONTACT_UPDATE |
| 35 | CONTACT_PRESENCE |
| 36 | CONTACT_LIST |
| 37 | CONTACT_SEARCH |
| 38 | CONTACT_MUTUAL |
| 39 | CONTACT_PHOTOS |
| 30 | CONTACT_SORT |
| 42 | CONTACT_VERIFY |
| 43 | REMOVE_CONTACT_PHOTO |
| 46 | CONTACT_INFO_BY_PHONE |
| 48 | CHAT_INFO |
| 49 | CHAT_HISTORY |
| 50 | CHAT_MARK |
| 51 | CHAT_MEDIA |
| 52 | CHAT_DELETE |
| 53 | CHATS_LIST |
| 54 | CHAT_CLEAR |
| 55 | CHAT_UPDATE |
| 56 | CHAT_CHECK_LINK |
| 57 | CHAT_JOIN |
| 58 | CHAT_LEAVE |
| 59 | CHAT_MEMBERS |
| 60 | PUBLIC_SEARCH |
| 61 | CHAT_PERSONAL_CONFIG |
| 63 | CHAT_CREATE |
| 64 | MSG_SEND |
| 65 | MSG_TYPING |
| 66 | MSG_DELETE |
| 67 | MSG_EDIT |
| 68 | CHAT_SEARCH |
| 70 | MSG_SHARE_PREVIEW |
| 71 | MSG_GET |
| 72 | MSG_SEARCH_TOUCH |
| 73 | MSG_SEARCH |
| 74 | MSG_GET_STAT |
| 75 | CHAT_SUBSCRIBE |
| 76 | VIDEO_CHAT_START |
| 77 | CHAT_MEMBERS_UPDATE |
| 78 | VIDEO_CHAT_START_ACTIVE |
| 79 | VIDEO_CHAT_HISTORY |
| 80 | PHOTO_UPLOAD |
| 81 | STICKER_UPLOAD |
| 82 | VIDEO_UPLOAD |
| 83 | VIDEO_PLAY |
| 84 | VIDEO_CHAT_CREATE_JOIN_LINK |
| 86 | CHAT_PIN_SET_VISIBILITY |
| 87 | FILE_UPLOAD |
| 88 | FILE_DOWNLOAD |
| 89 | LINK_INFO |
| 92 | MSG_DELETE_RANGE |
| 96 | SESSIONS_INFO |
| 97 | SESSIONS_CLOSE |
| 98 | PHONE_BIND_REQUEST |
| 99 | PHONE_BIND_CONFIRM |
| 103 | GET_INBOUND_CALLS |
| 104 | AUTH_2FA_DETAILS |
| 105 | EXTERNAL_CALLBACK |
| 107 | AUTH_VALIDATE_PASSWORD |
| 108 | AUTH_VALIDATE_HINT |
| 109 | AUTH_VERIFY_EMAIL |
| 110 | AUTH_CHECK_EMAIL |
| 111 | AUTH_SET_2FA |
| 112 | AUTH_CREATE_TRACK |
| 113 | AUTH_CHECK_PASSWORD |
| 115 | AUTH_LOGIN_CHECK_PASSWORD |
| 116 | AUTH_LOGIN_PROFILE_DELETE |
| 101 | AUTH_LOGIN_RESTORE_PASSWORD |
| 117 | CHAT_COMPLAIN |
| 118 | MSG_SEND_CALLBACK |
| 119 | SUSPEND_BOT |
| 124 | LOCATION_STOP |
| 125 | LOCATION_SEND |
| 126 | LOCATION_REQUEST |
| 127 | GET_LAST_MENTIONS |
| 128 | NOTIF_MESSAGE |
| 129 | NOTIF_TYPING |
| 130 | NOTIF_MARK |
| 131 | NOTIF_CONTACT |
| 132 | NOTIF_PRESENCE |
| 134 | NOTIF_CONFIG |
| 135 | NOTIF_CHAT |
| 136 | NOTIF_ATTACH |
| 137 | NOTIF_CALL_START |
| 139 | NOTIF_CONTACT_SORT |
| 140 | NOTIF_MSG_DELETE_RANGE |
| 142 | NOTIF_MSG_DELETE |
| 143 | NOTIF_CALLBACK_ANSWER |
| 144 | CHAT_BOT_COMMANDS |
| 145 | BOT_INFO |
| 147 | NOTIF_LOCATION |
| 148 | NOTIF_LOCATION_REQUEST |
| 150 | NOTIF_ASSETS_UPDATE |
| 152 | NOTIF_DRAFT |
| 153 | NOTIF_DRAFT_DISCARD |
| 154 | NOTIF_MSG_DELAYED |
| 155 | NOTIF_MSG_REACTIONS_CHANGED |
| 156 | NOTIF_MSG_YOU_REACTED |
| 158 | OK_TOKEN |
| 159 | NOTIF_PROFILE |
| 160 | WEB_APP_INIT_DATA |
| 161 | COMPLAIN |
| 162 | COMPLAIN_REASONS_GET |
| 176 | DRAFT_SAVE |
| 177 | DRAFT_DISCARD |
| 178 | MSG_REACTION |
| 179 | MSG_CANCEL_REACTION |
| 180 | MSG_GET_REACTIONS |
| 181 | MSG_GET_DETAILED_REACTIONS |
| 193 | STICKER_CREATE |
| 194 | STICKER_SUGGEST |
| 195 | VIDEO_CHAT_MEMBERS |
| 196 | CHAT_HIDE |
| 198 | CHAT_SEARCH_COMMON_PARTICIPANTS |
| 199 | PROFILE_DELETE |
| 200 | PROFILE_DELETE_TIME |
| 257 | CHAT_REACTIONS_SETTINGS_SET |
| 258 | REACTIONS_SETTINGS_GET_BY_CHAT_ID |
| 259 | ASSETS_REMOVE |
| 260 | ASSETS_MOVE |
| 261 | ASSETS_LIST_MODIFY |
| 272 | FOLDERS_GET |
| 273 | FOLDERS_GET_BY_ID |
| 274 | FOLDERS_UPDATE |
| 275 | FOLDERS_REORDER |
| 276 | FOLDERS_DELETE |
| 277 | NOTIF_FOLDERS |
| 288 | Запрос QR-кода для входа |
| 289 | Статус QR-кода по trackId |
| 290 | AUTH_QR_APPROVE |
| 291 | Вход по trackId после сканирования QR-кода |
| 292 | NOTIF_BANNERS |
