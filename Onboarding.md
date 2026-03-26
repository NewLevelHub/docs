# Onboarding — NewLevelHub

Привет! Эта страница для тех кто только пришёл в команду.  
Прочитай её полностью — займёт 15 минут, сэкономит несколько дней.

---

## 1. Что такое NewLevelHub

> Сюда напишите 2-3 предложения о продукте — что делает, для кого, какая цель.

---

## 2. Команда

| Роль | Имя | Контакт |
|---|---|---|
| Tech Lead | — | — |
| Tech Lead | — | — |
| Fullstack | — | — |
| Fullstack | — | — |
| QA | — | — |

---

## 3. Инструменты — получи доступы в первый день

- [ ] GitHub org: https://github.com/newlevelhub
- [ ] Linear: https://linear.app/newlevelhub
- [ ] Staging: https://staging.newlevelhub.com
- [ ] Slack / мессенджер команды
- [ ] .env файлы (попроси у тимлида)

---

## 4. Что установить локально

```bash
# Python (бэкенд)
python 3.11+
poetry

# Node.js (фронтенд)
node 18+
npm

# Go (IoT)
go 1.21+

# Общее
git
docker
docker-compose
```

---

## 5. Как запустить проект

Смотри [Getting Started](Getting-Started)

---

## 6. Как работает процесс разработки

1. Берёшь тикет в Linear со статусом **Ready**
2. Создаёшь ветку от `main`: `git checkout -b feature/TICKET-123-description`
3. Открываешь **Draft PR** сразу — Linear автоматом ставит **In Progress**
4. Кодишь, коммитишь атомарно (Conventional Commits)
5. Готово → переводишь PR в **Ready for Review**, назначаешь ревьюера
6. После Approve мержишь в `main`
7. Заполняешь чеклист → статус **Ready for QA**
8. QA тестирует → **Done**

---

## 7. Правила коммитов

```
feat(scope): что добавил
fix(scope): что исправил
chore: что обновил
docs: обновил документацию
refactor: рефакторинг без новой функциональности
```

---

## 8. Чеклист Ready for QA

Перед тем как ставить статус Ready for QA убедись:

- [ ] PR смержен в `main`
- [ ] Staging обновился и работает
- [ ] AC заполнены в тикете Linear
- [ ] Документация обновлена в Wiki
- [ ] Label `Ready for QA` добавлен в PR

---

## 9. Полезные ссылки

- [Architecture](Architecture) — как устроена система
- [Backend](Backend/Overview) — Python сервис
- [Frontend](Frontend/Overview) — веб-клиент
- [IoT](IoT/Overview) — Go сервис
- [AI](AI/Overview) — ML модели
