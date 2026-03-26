# Deployment — Overview

Как деплоить сервисы NewLevelHub.

---

## Окружения

| Окружение | Ветка | URL | Деплой |
|---|---|---|---|
| Staging | `main` | https://staging.newlevelhub.com | Автоматически при мерже в main |
| Production | `main` (тег) | https://newlevelhub.com | Вручную |

---

## Staging — автоматический деплой

При мерже PR в `main` staging обновляется автоматически.  
Проверь что staging поднялся перед тем как ставить **Ready for QA**.

---

## Чеклист перед деплоем на Prod

- [ ] QA принял все тикеты в релизе
- [ ] Миграции БД проверены
- [ ] .env переменные на проде актуальны
- [ ] Сделан бэкап БД
- [ ] Команда предупреждена

---

## Страницы

- [Backend Deployment](Deployment/Backend)
- [Frontend Deployment](Deployment/Frontend)
- [IoT Deployment](Deployment/IoT)
