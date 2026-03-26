# Backend — API Endpoints

Все эндпоинты NewLevelHub API.  
Swagger UI (локально): http://localhost:8000/docs  
Swagger UI (staging): https://staging.newlevelhub.com/docs

---

## Авторизация

Все защищённые эндпоинты требуют заголовок:
```
Authorization: Bearer <access_token>
```

---

## Auth

| Метод | Путь | Auth | Описание |
|---|---|---|---|
| POST | `/api/v1/auth/register` | Нет | Регистрация |
| POST | `/api/v1/auth/login` | Нет | Авторизация |
| POST | `/api/v1/auth/refresh` | Нет | Обновить токен |
| POST | `/api/v1/auth/logout` | Да | Выход |

---

<!-- 
  Добавляй новые разделы по мере разработки.
  Формат:
  
  ## Название модуля
  
  | Метод | Путь | Auth | Описание |
  |---|---|---|---|
  | GET | `/api/v1/...` | Да/Нет | Описание |
-->
