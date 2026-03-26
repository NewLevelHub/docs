# Backend — Overview

Python сервис NewLevelHub. Основной API для фронтенда и внешних сервисов.

---

## Стек

| Технология | Версия | Описание |
|---|---|---|
| Python | 3.11+ | Язык |
| FastAPI | — | Web framework |
| PostgreSQL | — | Основная БД |
| Redis | — | Кэш, очереди |
| Poetry | — | Управление зависимостями |
| Docker | — | Контейнеризация |

---

## Структура проекта

```
backend/
├── app/
│   ├── api/          ← роутеры (эндпоинты)
│   ├── core/         ← конфиг, безопасность
│   ├── models/       ← модели БД
│   ├── schemas/      ← Pydantic схемы
│   ├── services/     ← бизнес-логика
│   └── utils/        ← вспомогательные функции
├── tests/
├── .env.example
├── docker-compose.yml
├── pyproject.toml
└── main.py
```

---

## Страницы

- [Setup](Backend/Setup) — как запустить локально
- [API Endpoints](Backend/API-Endpoints) — все эндпоинты
- [Modules](Backend/Modules) — описание модулей
