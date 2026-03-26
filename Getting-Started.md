# Getting Started — NewLevelHub

Как запустить весь проект локально с нуля.

---

## Требования

- Python 3.11+
- Node.js 18+
- Go 1.21+
- Docker + docker-compose
- Git

---

## 1. Клонируй репозитории

```bash
git clone git@github.com:newlevelhub/backend.git
git clone git@github.com:newlevelhub/frontend.git
git clone git@github.com:newlevelhub/iot.git
git clone git@github.com:newlevelhub/ai.git
```

---

## 2. Запусти инфраструктуру (БД, Redis)

```bash
cd backend
docker-compose up -d
```

---

## 3. Запусти Backend

```bash
cd backend
cp .env.example .env      # заполни переменные
poetry install
poetry run python main.py
```

Backend доступен на http://localhost:8000  
Swagger docs: http://localhost:8000/docs

---

## 4. Запусти Frontend

```bash
cd frontend
cp .env.example .env
npm install
npm run dev
```

Frontend доступен на http://localhost:3000

---

## 5. Запусти IoT сервис

```bash
cd iot
cp .env.example .env
go mod tidy
go run main.go
```

---

## Переменные окружения

Файлы `.env.example` есть в каждом репо.  
Реальные значения для локальной разработки — попроси у тимлида.

---

## Частые проблемы

| Проблема | Решение |
|---|---|
| БД не поднимается | Проверь что Docker запущен |
| Port already in use | Убей процесс: `lsof -ti:8000 \| xargs kill` |
| — | — |
