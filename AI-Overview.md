# AI — Overview

ML сервис NewLevelHub. Собственные модели, обучение и инференс.

---

## Стек

| Технология | Описание |
|---|---|
| Python 3.11+ | Язык |
| — | ML Framework (PyTorch / TensorFlow / sklearn) |
| — | Хранилище моделей |
| Docker | Контейнеризация |

---

## Структура проекта

```
ai/
├── models/           ← сохранённые модели
├── training/         ← скрипты обучения
├── inference/        ← инференс сервис
├── data/             ← датасеты (gitignore)
├── notebooks/        ← Jupyter notebooks
├── .env.example
└── requirements.txt
```

---

## Страницы

- [Models](AI/Models) — описание каждой модели
- [Training](AI/Training) — как обучать, датасеты, метрики
- [Inference](AI/Inference) — как использовать модели в продакшне
