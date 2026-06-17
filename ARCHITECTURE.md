# ARCHITECTURE — карта проекта

## Назначение
Каталог переиспользуемых настроек для проектов Claude Code. Каждая настройка — отдельная
карточка-`.md`. При создании нового проекта проходим по каталогу и выбираем, что добавить.

## Структура
```
general project settings/
├── README.md                      # как пользоваться проектом
├── MEMORY.md                      # журнал (статус, открытые вопросы, лог сессий)
├── ARCHITECTURE.md                # этот файл: карта + changelog
├── project-starter.md             # базовый свод правил работы для новых проектов
├── .gitignore                     # исключает .claude/ (машинно-зависимое)
└── settings/
    ├── README.md                  # индекс каталога (таблица настроек)
    └── notification-sound.md      # карточка: звук, когда Claude ждёт ответа
```

## Ключевые файлы
- `settings/README.md` — индекс. С него начинается обход каталога.
- `project-starter.md` — соглашения по работе, включая `git save`/`git load` (раздел 6).

## Команды (воркфлоу)
- **`git load`** — в начале сессии: прочитать `MEMORY.md` + ключевые файлы, кратко доложить
  статус. Только чтение.
- **`git save`** — в конце блока работы: обновить `MEMORY.md` (запись сверху) и changelog ниже,
  `git add -A`, коммит; push — по команде пользователя.

## Changelog
### 2026-06-17
- Создан каталог настроек: `settings/README.md`, `settings/notification-sound.md`.
- Заведены контекстные файлы `MEMORY.md`, `ARCHITECTURE.md`, `README.md` (воркфлоу git save/load).
- git init + remote `konicaRu/general-project-settings`; `.gitignore` исключает `.claude/`.
- Первый push на GitHub (`git save`).
