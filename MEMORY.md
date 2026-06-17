# MEMORY — журнал

## Статус проекта
Каталог общих настроек для проектов Claude Code. Цель: собирать лучшие настройки карточками
и при старте нового проекта проходить по ним, выбирая нужные.
- Git: репозиторий инициализирован, remote `https://github.com/konicaRu/general-project-settings`, ветка `main`.
- Запушено на GitHub.

## Открытые вопросы
- (нет открытых)

## Лог сессий
### 2026-06-17 (старт каталога + звук-уведомление + git-воркфлоу)
- Создан каталог: `settings/README.md` (индекс) + `settings/notification-sound.md` (карточка).
- Настроено и проверено звуковое уведомление, когда Claude ждёт ответа (глобальный хук во всех
  проектах). Два урока: нужны события `Notification`+`PermissionRequest`+`Elicitation`; играть
  mp3 из фонового хука можно только через `winmm.dll`/`mciSendString` (не `WMPlayer`/`MediaPlayer`).
- Заведён git-воркфлоу `git save`/`git load`: добавлены `MEMORY.md`, `ARCHITECTURE.md`, `README.md`.
- `.gitignore` исключает машинно-зависимый `.claude/`.
- Выполнен `git save`: каталог запушен на GitHub (`konicaRu/general-project-settings`).
