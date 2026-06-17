# Звуковое уведомление, когда Claude ждёт ответа

**Тип:** глобальный хук Claude Code (`Notification`)
**Область:** все проекты сразу (файл `~/.claude/settings.json`)
**Что делает:** когда появляется окно, требующее твоего ответа или разрешения, проигрывается звуковой сигнал.

---

## Зачем

Claude иногда останавливается и ждёт твоего ввода (вопрос, запрос разрешения на действие). Легко это пропустить, если переключился на другое окно. Хук проигрывает звук — ты сразу слышишь, что Claude ждёт.

---

## Как это устроено

Две части:

1. **Хук** в `~/.claude/settings.json` — ловит событие `Notification` и запускает PowerShell-скрипт в фоне (`async`).
2. **Скрипт** `~/.claude/play-notification.ps1` — проигрывает mp3 через `System.Windows.Media.MediaPlayer`.

Звук лежит здесь: `C:\claude_code_projects\error1-notification.mp3`

---

## Установка в новый проект

Это **глобальная** настройка — она уже действует во всех проектах, ставить отдельно в проект не нужно. Ниже — на случай переустановки системы или нового компьютера.

### 1. Блок в `~/.claude/settings.json` (раздел `hooks`)

```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File \"C:\\Users\\<ИМЯ>\\.claude\\play-notification.ps1\"",
            "async": true,
            "statusMessage": "Notification sound"
          }
        ]
      }
    ]
  }
}
```

### 2. Скрипт `~/.claude/play-notification.ps1`

```powershell
Add-Type -AssemblyName presentationCore
$p = New-Object System.Windows.Media.MediaPlayer
$p.Open([uri]'C:\claude_code_projects\error1-notification.mp3')
Start-Sleep -Milliseconds 400
$p.Play()
Start-Sleep -Seconds 3
$p.Stop()
$p.Close()
```

### 3. Положить звук

Скопировать `error1-notification.mp3` в `C:\claude_code_projects\` (или поменять путь в скрипте на свой).

---

## Проверка

Запустить скрипт вручную — должен прозвучать сигнал:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Users\<ИМЯ>\.claude\play-notification.ps1"
```

Если правил `settings.json` в уже открытой сессии — открой меню `/hooks` один раз (перечитывает конфиг) или перезапусти Claude Code.

---

## Настройка под себя

- **Другой звук:** поменяй путь в строке `$p.Open([uri]'...')`.
- **Длиннее/короче:** меняй `Start-Sleep -Seconds 3` (длительность проигрывания).
- **Свой mp3:** формат любой, который понимает Windows Media (mp3, wav, wma).
