# IoT — Overview

Go сервис NewLevelHub. Получает данные с IoT устройств и передаёт в Backend.

---

## Стек

| Технология | Описание |
|---|---|
| Go 1.21+ | Язык |
| — | Протокол (MQTT / HTTP / WebSocket) |
| Docker | Контейнеризация |

---

## Структура проекта

```
iot/
├── cmd/
│   └── main.go
├── internal/
│   ├── handlers/     ← обработчики сообщений
│   ├── models/       ← структуры данных
│   └── services/     ← бизнес-логика
├── .env.example
├── go.mod
└── Dockerfile
```

---

## Data Flow

```
IoT устройство
     │
     │ (протокол: ?)
     ▼
IoT Service (Go)
     │
     │ HTTP / gRPC
     ▼
Backend (Python)
     │
     ▼
PostgreSQL
```

---

## Страницы

- [Setup](IoT/Setup) — как запустить локально
- [Devices](IoT/Devices) — описание устройств и протоколов
- [Data Flow](IoT/Data-Flow) — детальная схема потока данных
