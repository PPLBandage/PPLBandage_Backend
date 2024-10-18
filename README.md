# PPLBandage Backend
<p align="left">
    <img src="https://made-with.prisma.io/indigo.svg" alt='prisma'></img> 
    <img src="https://img.shields.io/endpoint?url=https%3A%2F%2Fghloc.vercel.app%2Fapi%2FPPLBandage%2FPPLBandage_Backend%2Fbadge%3Ffilter%3D.ts%24%2C.tsx%24%2C.css%24" alt='prisma'></img> 
    <img src="https://andcool.ru/static/badges/made-for-ppl.svg" alt='pepeland'></img> 
</p>

## Оглавление
- [Base URL](#base-url)
- [Ограничения по частоте запросов](#ограничения-по-частоте-запросов)
- [Ошибки](#ошибки)
- [Аутентификация](#аутентификация)
- Документация к API
- - [Корневые эндпоинты API](/docs/mainRoute.md)
- - [Мастерская](/docs/workshop.md)
- - Пользователи
- - [Minecraft API](/docs/minecraft.md)
- - Auth

## Введение
**API PPLBandage — это RestFul API, позволяющий приложениям взаимодействовать с серверами и базой данных проекта.**  

## Base URL
`https://pplbandage.ru/api/v1`

## Ограничения по частоте запросов
Глобальное ограничение на все эндпоинты: `150rpm`, однако оно может варьироваться в зависимости от конечного эндпоинта.
| Эндпоинт                                         | Ограничение по частоте запросов |
| ------------------------------------------------ | ------------------------------- |
| `GET /ping`                                      | Нет                             |
| `GET /user/me/connections/minecraft/cache/purge` | `5rpm`                          |
| `POST /workshop`                                 | `5rpm`                          |
| `GET /workshop/:id`                              | Нет                             |
| `GET /workshop/:id/info`                         | Нет                             |

Отсутствие ограничения по количеству запросов на эндпоинтах означает, что они являются закрытыми и для доступа к ним нужен специальный ключ. Об этом подробнее в описании эндпоинтов.

## Ошибки
Всего может возникнуть несколько типов ошибок:
### Ошибки валидации
Ошибки валидации могут возникнуть, если в тело запроса или query параметры было передано неверное значение.
```json
{
    "message": [
        "Массив ошибок валидации"
    ],
    "error": "Описание кода ошибки",
    "statusCode": 400
}
```

### Общие ошибки
Общие ошибки могут возникнуть во всех остальных случаях, например, если пользователь не авторизован.
```json
{
    "statusCode": 401,
    "message": "UNAUTHORIZED"
}
```
Так же в ошибках может быть поле `message_ru`, содержащее описание ошибки на русском.

## Аутентификация
Аутентификация происходит через **токены сессии**. 
### Описание процесса аутентификации
Токены сессии должны передаваться в cookies запроса по ключу `sessionId`.  
При создании запроса к API он проверяет валидность токена, время жизни которого равно `14 дням` или двум неделям. Если со времени создания токена прошло более половины от его времени жизни, то бекенд обновляет токен и отправляет клиенту в `Setcookie` хедере.
> [!NOTE]
> Обычно, все эндпоинты, требующие строгой аутентификации всегда отправляют `Setcookie` хедер, содержащий актуальный в данный момент токен сессии. В случае, когда токен был обновлён, он так же отправляется в хедерах. Клиенты должны при каждом запросе устанавливать актуальный токен из запроса.

> [!WARNING]
> Бекенд **не поддерживает** выполнение конкурентных запросов при обновлении токена. Клиенты должны сами позаботиться, чтобы запросы, подразумевающие обновление токена не выполнялись параллельно. В ином случае есть риск потерять текущую сессию.

Если время жизни токена закончилось, бекенд вернёт HTTP код `401`.  
Все сессии связываются с fingerprint'ом текущего клиента, который представлен `User-Agent` хедером. Если запрос делается с другого `User-Agent` и по этому же токену, то он автоматически удаляется, возвращая HTTP код `401`.