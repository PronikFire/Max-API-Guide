# Гайд на API MAX

### Нашли ошибку или хотите задать вопрос? Создайте *Issue*

## Общая информация

У Max существует два API:

* **WSS (WebSocket)** - для web-версии
* **TLS** - для приложений

По факту это один и тот же API, но разница между ними все же есть. Как пример: в web-версии вырезали аутентификацию по номеру телефона.

---

## Как анализировать Web API

Анализ можно проводить прямо в браузере с помощью панели разработчика:

1. Заходим на сайт: [web.max.ru](https://web.max.ru)
2. Открываем панель разработчика:
* `Ctrl + Shift + J`, либо
* ПКМ -> **Проверить** (название пункта зависит от браузера)


3. Переходим во вкладку **Network** (Сеть)
4. *(Опционально)* В фильтрах сверху выбираем **WS** (или **Socket**)
5. Перезагружаем страницу
6. Находим и открываем WebSocket-подключение:
* Если выполнен пункт 4, оно будет единственным в списке


7. Переходим во вкладку **Messages** (Сообщения)

После выбора сообщения в левом нижнем углу можно изменить режим отображения **View** (по умолчанию там выставлено UTF-8).

Про содержание сообщений подробнее написано [здесь](#содержание-сообщений).

---

## Как анализировать App API

Здесь не обойтись без стороннего софта.

1. Скачиваем Windows-версию приложения:
* https://download.max.ru/#desktop


2. Устанавливаем:
* *Рекомендуется использовать виртуальную машину, но это не принципиально.*


3. Скачиваем и устанавливаем **mitmproxy**:
* https://www.mitmproxy.org


4. Запускаем **mitmweb** (веб-интерфейс, так как он более удобен)
5. Устанавливаем сертификат:
* Путь к файлу: `C:\Users\%User%\.mitmproxy\mitmproxy-ca-cert.cer`
* Устанавливать для: **Локального компьютера**
* Хранилище: **Доверенные корневые центры сертификации**



### Настройка mitmproxy

В интерфейсе mitmproxy:

1. Переходим во вкладку **Capture**
2. Включаем **Local Applications**
3. Выбираем `max.exe`
* *Max должен быть запущен в фоне.*


4. Переходим во вкладку **Flow List**
5. Очищаем список:
* **File -> Clear All**


6. *(Опционально)* Перезапускаем Max.

Готово.

После выбора соединения в правом верхнем углу можно выбрать режим **View**.

Наиболее полезные режимы:

* **Hex Dump**
* **Hex Stream** - то же самое, но в одну строку

Содержание сообщений смотрите далее.

---

## Содержание сообщений

Заголовок составляет 10 байт.

| Номер байта | Описание |
| --- | --- |
| 1 | `ver` - версия протокола |
| 2 | `cmd` - команда (см. [значения cmd](#значения-cmd)) |
| 3..4 | `seq` - порядковый номер операции. |
| 5..6 | `opcode` - код операции (см. [список opcode](#список-opcode)) |
| 7 | `cof` - степень сжатия LZ4 (`0` - сжатие не используется) |
| 8..10 | Длина payload в байтах |
| 11.. | `Payload` - содержание сообщения (кодируется в формате **MsgPack**) |

### Советы по анализу сообщений

Для анализа **Hex Stream** удобно использовать:

* [hexed.it](https://hexed.it) - отличный онлайн-инструмент для работы с бинарными данными.
* [HxD](https://mh-nexus.de/en/hxd/) - полноценный офлайн-аналог.

Также советую декомпилировать мобильную версию приложения с помощью **jadx**. Это даст гораздо больше понимания внутренних процессов.

---

## Значения `cmd`

| Значение | Описание |
| --- | --- |
| 0 | Request |
| 1 | Response |
| 3 | Error |

---

## Сжатие

Байт `cof` отвечает за коэффициент сжатия.


Формула расчета: $\lfloor \text{исходная длина} / \text{длина при сжатии} \rfloor + 1$ (округляется до целого).


Сжатие применяется только в том случае, если длина payload превышает 32 байта.

---

## Список `opcode`

Здесь собраны все возможные opcode. \
Актуально для Android-приложения версии `26.28.0` и версии протокола: `10`.
Знак `~` означает, что opcode не подтвержден исходниками выбранной версии приложения.


| Opcode | Описание |
| --- | --- |
| 1 | [PING](#ping) |
| 2 | [DEBUG](#debug) |
| 3 | [RECONNECT](#reconnect) |
| 5 | [LOG](#log) |
| 6 | [SESSION_INIT](#session_init) |
| 8 | [LOGIN2](#login2) |
| 16 | [PROFILE](#profile) |
| 17 | [AUTH_REQUEST](#auth_request) |
| 18 | [AUTH](#auth) |
| 19 | [LOGIN](#login) |
| 20 | [LOGOUT](#logout) |
| 21 | [SYNC](#sync) |
| 22 | [CONFIG](#config) |
| 23 | [AUTH_CONFIRM](#auth_confirm) |
| 25 | [PRESET_AVATARS](#preset_avatars) |
| 26 | [ASSETS_GET](#assets_get) |
| 27 | [ASSETS_UPDATE](#assets_update) |
| 28 | [ASSETS_GET_BY_IDS](#assets_get_by_ids) |
| 29 | [ASSETS_ADD](#assets_add) |
| 32 | [CONTACT_INFO](#contact_info) |
| 33 | CONTACT_ADD |
| 34 | [CONTACT_UPDATE](#contact_update) |
| 35 | [CONTACT_PRESENCE](#contact_presence) |
| 36 | [CONTACT_LIST](#contact_list) |
| 37 | [CONTACT_SEARCH](#contact_search) |
| 38 | [CONTACT_MUTUAL](#contact_mutual) ~ |
| 39 | [CONTACT_PHOTOS](#contact_photos) |
| 40 | CONTACT_SORT |
| 42 | [CONTACT_VERIFY](#contact_verify) |
| 43 | [REMOVE_CONTACT_PHOTO](#remove_contact_photo) |
| 46 | [CONTACT_INFO_BY_PHONE](#contact_info_by_phone) |
| 48 | [CHAT_INFO](#chat_info) |
| 49 | [CHAT_HISTORY](#chat_history) |
| 50 | [CHAT_MARK](#chat_mark) |
| 51 | [CHAT_MEDIA](#chat_media) |
| 52 | [CHAT_DELETE](#chat_delete) |
| 53 | [CHATS_LIST](#chats_list) |
| 54 | [CHAT_CLEAR](#chat_clear) |
| 55 | [CHAT_UPDATE](#chat_update) |
| 56 | [CHAT_CHECK_LINK](#chat_check_link) |
| 57 | [CHAT_JOIN](#chat_join) |
| 58 | [CHAT_LEAVE](#chat_leave) |
| 59 | [CHAT_MEMBERS](#chat_members) |
| 60 | [PUBLIC_SEARCH](#public_search) |
| 61 | [CHAT_PERSONAL_CONFIG](#chat_personal_config) |
| 62 | CHAT_LIVESTREAM_INFO |
| 63 | CHAT_CREATE ~ |
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
| 91 | GET_COMMENTS_UPDATES |
| 92 | MSG_DELETE_RANGE |
| 94 | MSG_DELETE_USER |
| 96 | SESSIONS_INFO |
| 97 | SESSIONS_CLOSE |
| 98 | PHONE_BIND_REQUEST |
| 99 | PHONE_BIND_CONFIRM |
| 101 | AUTH_LOGIN_RESTORE_PASSWORD |
| 103 | GET_INBOUND_CALLS |
| 104 | AUTH_2FA_DETAILS |
| 105 | EXTERNAL_CALLBACK |
| 106 | PHONE_WEBAPP_SHARE |
| 107 | AUTH_VALIDATE_PASSWORD |
| 108 | AUTH_VALIDATE_HINT |
| 109 | AUTH_VERIFY_EMAIL |
| 110 | AUTH_CHECK_EMAIL |
| 111 | AUTH_SET_2FA |
| 112 | AUTH_CREATE_TRACK |
| 113 | AUTH_CHECK_PASSWORD |
| 115 | AUTH_LOGIN_CHECK_PASSWORD |
| 116 | AUTH_LOGIN_PROFILE_DELETE |
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
| 152 | NOTIF_DRAFT ~ |
| 153 | NOTIF_DRAFT_DISCARD ~ |
| 154 | NOTIF_MSG_DELAYED |
| 155 | NOTIF_MSG_REACTIONS_CHANGED |
| 156 | NOTIF_MSG_YOU_REACTED |
| 158 | OK_TOKEN |
| 159 | NOTIF_PROFILE |
| 160 | WEB_APP_INIT_DATA |
| 161 | COMPLAIN |
| 162 | COMPLAIN_REASONS_GET |
| 163 | CALL_HISTORY |
| 164 | CALL_HISTORY_CLEAR |
| 165 | NOTIF_CALL_HISTORY |
| 166 | VIDEO_CHAT_JOIN |
| 176 | DRAFT_SAVE ~ |
| 177 | DRAFT_DISCARD ~ |
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
| 202 | TRANSCRIBE_MEDIA |
| 203 | PHOTO_URL_REFRESH |
| 208 | STORIES_LIST |
| 209 | STORIES_LIST_BY_OWNER_ID |
| 210 | STORIES_GET_BY_OWNER_ID |
| 211 | STORIES_GET_STATS |
| 212 | STORIES_GET_DETAILED_STATS |
| 213 | STORIES_REACT |
| 214 | STORIES_MARK |
| 215 | STORIES_SEND |
| 216 | NOTIF_STORIES_UPDATE |
| 217 | STORIES_EDIT |
| 218 | STORIES_DELETE |
| 220 | STORIES_GET_BY_STORY_ID |
| 256 | ORG_INFO |
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
| 290 | AUTH_QR_APPROVE |
| 292 | NOTIF_BANNERS |
| 293 | NOTIF_TRANSCRIPTION |
| 300 | CHAT_SUGGEST |
| 301 | AUDIO_PLAY |
| 302 | BANNERS_GET |
| 303 | MSG_DELIVERY |
| 304 | SEND_VOTE |
| 305 | VOTERS_LIST_BY_ANSWER |
| 306 | GET_POLL_UPDATES |
| 307 | CHAT_CHECK_ESIA |

---

## Структура сообщения ошибки
```
Error
{
	string description
	string error
	string title
	string message
	string localizedMessage
}
```
`title` и `description` не гарантированы.

---


## Структуры запросов и ответов

Здесь описаны структуры сообщений запросов и ответов под каждый opcode. Названия полей полностью соответствуют тем, что будут в сообщениях, т.е. их можно использовать для парсинга.


`Optional` не значит, что поле можно полностью игнорировать. Оно может быть обязательным при определённых условиях.\
`EnumAsString` значит, что поле представляет из себя String, но может содержать ограниченное количество значений, которые можно представить в виде Enum.

### PING
```
Request
{
	bool interactive
}
```
```
Response { }
```

### DEBUG
```
Request
{
	[EnumAsString]
	CmdType cmd
	string[] args
}
```
```
Response { }
```

### RECONNECT
```
Request
{
	bool tls
	string redirectHost
}
```
```
Response { }
```

### LOG
```
Request
{
	ApiLogEntry[] events
}
```
```
Response { }
```

### SESSION_INIT
```
Request 
{
	UserAgent userAgent
	string deviceId
	long clientSessionId
	[Optional]
	string mt_instanceid
}
```
```
Response
{
	long callsSeed
	bool isVpn
	string[] reg-country-code
	int app-update-type
	string location
	string recovery-url
}
```

### LOGIN2
```
Request
{
	string configHash
	long contactsSync
	bool needProfile
}
```
```
Response
{
	Configuration config
	Profile profile
	ContactInfo[] contacts
}
```

### PROFILE
```
Request
{
	[Optional]
	string firstName
	[Optional]
	string lastName
	[Optional]
	string photoToken
	[Optional]
	long photoId
	[Optional]
	RectF crop
	[Optional]
	string description
	[Optional]
	string link
	[EnumAsString]
	AvatarType avatarType
}
```
```
Response
{
	Profile profile
}
```

### AUTH_REQUEST
```
Request
{
	string phone
	[EnumAsString]
	AuthType type
	[Optional]
	byte[] mode
}
```
```
Response
{
	int codeLength
	long altActionDuration
	int requestCountLeft
	string token
	long requestMaxDuration

}
```

### AUTH
```
Request
{
	string token
	[Optional]
	string verifyCode
	string authTokenType
}
```
```
Response
{
	Profile profile
	Dictionary<string, TokenAttribute> tokenAttrs
	NeuroAvatarsPresetInfo[] presetAvatars
	PasswordChallenge passwordChallenge
}
```

### LOGIN
```
Request
{
	string token
	bool interactive
	[Optional]
	long chatsSync
	[Optional]
	long contactsSync
	long presenceSync
	[Optional]
	string configHash
	[Optional]
	long callsSync
	[Optional]
	long lastLogin
	[Optional]
	long draftsSync
	[Optional]
	long bannersSync
	[Optional]
	byte[] chatCacheFingerprint
	[Optional]
	byte[] chatsCountGroups
	ExpObject exp
}
```
```
Response
{
	bool videoChatHistory
	long chatMarker
	Configuration config
	DraftsNews drafts
	Dictionary<long, Presence> presence
	ContactInfo[] contacts
	Dictionary<long, Message[]> messages
	Profile profile
	int updates
	long time
	Call[] calls
	Chats[] chats
	string token
	Login2Flags login2Flags
	long resetAt
}
```

### LOGOUT
```
Request
{
	string pushToken
}
```
```
Response { }
```

### SYNC
```
Request
{
	Dictionary<string, ContactNameWrapper> contactList
}
```
```
Response
{
	ContactInfo[] contacts
	Dictionary<string, long> phones
}
```

### CONFIG
```
Request
{
	[Optional]
	string pushToken
	[Optional]
	long pushOptions
	[Optional]
	Configuration settings
	[Optional]
	bool reset
}
```
```
Response
{
	string hash
	ConfigurationUserSettings user
}
```

### AUTH_CONFIRM
```
Request
{
	string token
	[EnumAsString]
	LoginTokenType tokenType
	string firstName
	[Optional]
	string lastName
	[Optional]
	long photoId
	[Optional, EnumAsString]
	AvatarType avatarType
}
```
```
Response
{
	[EnumAsString]
	LoginTokenType tokenType
	string token
	Profile profile
}
```

### PRESET_AVATARS
```
Request { }
```
```
Response
{
	NeuroAvatarsPresetInfo[] presetAvatars
}
```

### ASSETS_GET
```
Request
{
	[Optional, EnumAsString]
	AssetType type
	[Optional]
	string sectionId
	long from
	int count
	[Optional]
	string query
}
```
```
Response
{
	long marker
	long[] stickers
	long[] stickerSets
	Background[] backgrounds
}
```

### ASSETS_UPDATE
```
Request
{
	[Optional, EnumAsString]
	AssetType type
	long sync
	[Optional]
	long chatId
	[Optional]
	long userId
}
```
```
Response
{
	Dictionary<long, long> animojiUpdates
	Dictionary<long, long> stickerSetsUpdates
	long sync
	Dictionary<long, long> stickersUpdates
	Section[] sections
	Dictionary<long, long> animojiSetUpdates
	string[] stickersOrder
}
```

### ASSETS_GET_BY_IDS
```
Request
{
	AssetType type
	long[] ids
}
```
```
Response
{
	Animoji[] animoji
	AnimojiSet[] animojiSets
	Sticker[] stickers
	StickerSet[] stickerSets
}
```

### ASSETS_ADD
```
Request
{
	AssetType type
	long[] ids
}
```
```
Response
{
	bool success
	long updateTime
}
```

### CONTACT_INFO
```
Request
{
	long[] contactIds
	[Optional]
	long chat_id
}
```
```
Response
{
	ContactInfo[] contacts
}
```

### CONTACT_UPDATE
```
Request
{
	long contactId
	[Optional, EnumAsString]
	ContactUpdateAction action
	[Optional]
	long firstName
	[Optional]
	long lastName
}
```
```
Response
{
	ContactInfo contact
}
```

### CONTACT_PRESENCE
```
Request
{
	long[] contactIds
	[Optional]
	long sync
}
```
```
Response
{
	Dictionary<long, Presence> presence
	long time
}
```

### CONTACT_LIST
```
Request
{
	[EnumAsString]
	StatusType status
	[Optional]
	int from
	[Optional]
	int count
}
```
```
Response
{
	ContactInfo[] contacts
}
```

### CONTACT_SEARCH
```
Request
{
}
```
```
Response
{
	ContactSearchResult[] result
	int total
}
```

### CONTACT_MUTUAL
```
Request
{
}
```
```
Response
{
	long[] contactIds
}
```

### CONTACT_PHOTOS
```
Request
{
	long contactId
	[Optional]
	int count
	[Optional]
	int from
}
```
```
Response
{
	long[] ids
	string[] urls
	int total
}
```

### CONTACT_VERIFY
```
Request { }
```
```
Response
{
	[EnumAsString]
	VerifyResultType verifyResult
	string name
}
```

### REMOVE_CONTACT_PHOTO
```
Request
{
	long photoId
}
```
```
Response
{
	Profile profile
}
```

### CONTACT_INFO_BY_PHONE
```
Request
{
	string phone
}
```
```
Response
{
	ContactInfo contact
}
```

### CHAT_INFO
```
Request
{
	long[] chatIds
}
```
```
Response
{
	Chat chat
	ContactInfo user
	Chat[] chats
}
```

### CHAT_HISTORY
```
Request
{
	long chatId
	[Optional]
	long postId
	long from
	int forward
	long forwardTime
	int backward
	long backwardTime
	bool getChat
	bool getMessages
	[Optional]
	string chatAccessToken
	string itemType
	bool interactive
}
```
```
Response
{
	Chat chat
	Message[] messages
	long[] messageIds
}
```

### CHAT_MARK
```
Request
{
	long chatId
	long mark
	[Optional]
	long messageId
	[EnumAsString]
	MarkType type
}
```
```
Response
{
	long mark
	int unread
	bool success
}
```

### CHAT_MEDIA
```
Request
{
	long chatId
	[Optional]
	long messageId
	[Optional, EnumAsString]
	AttachType[] attachTypes
	[Optional]
	int forward
	[Optional]
	int backward
}
```
```
Response
{
	long forward
	Message[] messages
	int pos
	int total
	long backward
}
```

### CHAT_DELETE
```
Request
{
	long chatId
	long lastEventTime
	bool forAll
}
```
```
Response { }
```

### CHATS_LIST
```
Request
{
	long marker
	int count
}
```
```
Response 
{
	long marker
	Chat[] chats
}
```

### CHAT_CLEAR
```
Request
{
	long chatId
	long lastEventTime
	bool forAll
}
```
```
Response { }
```

### CHAT_UPDATE
```
Request
{
	long chatId
	[Optional, EnumAsString]
	AccessType access
	[Optional]
	string link
	[Optional]
	bool revokePrivateLink
	[Optional]
	bool removeLink
	[Optional]
	string description
	[Optional]	
	Dictionary<string, bool> options
	[Optional]
	string theme
	[Optional]
	string photoToken
	[Optional]
	RectF crop
	[Optional]
	long pinMessageId
	[Optional]
	bool notifyPin
	[Optional]
	long changeOwnerId
}
```
```
Response 
{
	Chat chat
}
```

### CHAT_CHECK_LINK
```
Request
{
	string link
	[EnumAsString]
	LinkType linkType
}
```
```
Response { }
```

### CHAT_JOIN
```
Request
{
	string chatAccessToken
	string link
}
```
```
Response 
{
	Chat chat
}
```

### CHAT_LEAVE
```
Request
{
	long chatId
}
```
```
Response { }
```

### CHAT_MEMBERS
```
Request
{
	long chatId
	[Optional, EnumAsString]
	MemberType type
	[Optional]
	long marker
	[Optional]
	int count
	[Optional]
	string query
}
```
```
Response
{
	Member[] members
	long marker
}
```

### PUBLIC_SEARCH
```
Request
{
	string query
	int count
	[Optional]
	long marker
	[Optional]
	SearchType type
}
```
```
Response
{
	long marker
	SearchResult[] result
	string ucpQId
	int total
}
```

### CHAT_PERSONAL_CONFIG
```
Request
{
	long chatId
	bool hideNonContactBar
}
```
```
Response
{
	Chat chat
}
```
