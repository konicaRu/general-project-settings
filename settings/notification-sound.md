# Звуковое уведомление, когда Claude ждёт ответа

**Тип:** глобальные хуки Claude Code (`Notification`, `PermissionRequest`, `Elicitation`)
**Область:** все проекты сразу (файл `~/.claude/settings.json`)
**Что делает:** когда появляется окно, требующее твоего ответа или разрешения, проигрывается звуковой сигнал.
**Статус:** проверено end-to-end — звук срабатывает на окнах-вопросах и запросах разрешения.

---

## Зачем

Claude иногда останавливается и ждёт ввода (вопрос с вариантами, запрос разрешения на действие). Легко пропустить, если переключился на другое окно. Хук проигрывает звук — сразу слышно, что Claude ждёт.

---

## Два важных урока (почему «в лоб» не работает)

1. **Одного события `Notification` мало.** Оно НЕ ловит окна-вопросы (`AskUserQuestion`) и авто-разрешаемые команды. Нужны ещё:
   - `PermissionRequest` — срабатывает перед окном запроса разрешения;
   - `Elicitation` — срабатывает на диалоги-вопросы с вариантами.
2. **`WMPlayer` COM и `System.Windows.Media.MediaPlayer` НЕ звучат** из фонового скрытого PowerShell, которым хук запускает скрипт (нужен message-pump/окно, которых там нет). Рабочий способ — **`winmm.dll` / `mciSendString`**: проигрывает mp3 из любого неинтерактивного контекста.

---

## Как это устроено

1. **Хуки** в `~/.claude/settings.json` на три события — каждый запускает один и тот же скрипт в фоне (`async`).
2. **Скрипт** `~/.claude/play-notification.ps1` — проигрывает mp3 через `mciSendString` и пишет строку в лог `C:\claude_code_projects\notification-hook.log` (чтобы можно было убедиться, что хук сработал).

Звук: `C:\claude_code_projects\error1-notification.mp3`

---

## Установка в новый проект

Это **глобальная** настройка — действует во всех проектах сразу, в проект отдельно ставить не нужно. Ниже — для переустановки системы / нового компьютера.

### 1. Раздел `hooks` в `~/.claude/settings.json`

Один и тот же блок под тремя событиями (подставь своё имя пользователя в путь):

```json
{
  "hooks": {
    "Notification":      [ { "hooks": [ { "type": "command", "command": "powershell.exe -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File \"C:\\Users\\<ИМЯ>\\.claude\\play-notification.ps1\"", "async": true, "statusMessage": "Notification sound" } ] } ],
    "PermissionRequest": [ { "hooks": [ { "type": "command", "command": "powershell.exe -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File \"C:\\Users\\<ИМЯ>\\.claude\\play-notification.ps1\"", "async": true, "statusMessage": "Notification sound" } ] } ],
    "Elicitation":       [ { "hooks": [ { "type": "command", "command": "powershell.exe -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File \"C:\\Users\\<ИМЯ>\\.claude\\play-notification.ps1\"", "async": true, "statusMessage": "Notification sound" } ] } ]
  }
}
```

### 2. Скрипт `~/.claude/play-notification.ps1`

```powershell
$sound = 'C:\claude_code_projects\error1-notification.mp3'
$log   = 'C:\claude_code_projects\notification-hook.log'

try { Add-Content -Path $log -Value ("{0}  hook fired" -f (Get-Date -Format 'yyyy-MM-dd HH:mm:ss')) } catch {}

try {
  Add-Type @"
using System.Text;
using System.Runtime.InteropServices;
public class WinMM {
  [DllImport("winmm.dll", CharSet=CharSet.Auto)]
  public static extern int mciSendString(string command, StringBuilder ret, int retLen, System.IntPtr hwndCallback);
}
"@
  $alias = "snd" + [guid]::NewGuid().ToString("N")
  [WinMM]::mciSendString("open `"$sound`" type mpegvideo alias $alias", $null, 0, [System.IntPtr]::Zero) | Out-Null
  [WinMM]::mciSendString("play $alias wait", $null, 0, [System.IntPtr]::Zero) | Out-Null
  [WinMM]::mciSendString("close $alias", $null, 0, [System.IntPtr]::Zero) | Out-Null
} catch {
  try { [console]::beep(880, 400) } catch {}
  try { Add-Content -Path $log -Value ("  play FAILED: {0}" -f $_) } catch {}
}
```

### 3. Положить звук

Скопировать `error1-notification.mp3` в `C:\claude_code_projects\` (или поменять путь `$sound` в скрипте).

---

## Проверка

1. Запустить скрипт так же, как его дёргает хук — должен прозвучать сигнал:
   ```powershell
   powershell.exe -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File "C:\Users\<ИМЯ>\.claude\play-notification.ps1"
   ```
2. Проверить, что хук реально срабатывает на окна: в логе `C:\claude_code_projects\notification-hook.log` появляются строки `... hook fired` в момент появления окна-вопроса/запроса разрешения.
3. Если правил `settings.json` в уже открытой сессии и звука нет — открой `/hooks` один раз (Claude перечитает конфиг) или перезапусти Claude Code.

> Скрипт менять можно прямо в открытой сессии без перезапуска — он вызывается заново каждый раз. Перезапуск/`/hooks` нужен только при изменении **списка событий** в `settings.json`.

---

## Настройка под себя

- **Другой звук:** поменяй путь в `$sound`. Формат — любой, что понимает MCI (mp3, wav, wma).
- **Громкость/длительность:** управляется самим файлом; `play ... wait` играет его целиком.
- **Без лога:** убери строку `Add-Content ... hook fired` (лог нужен только для диагностики).
- **Не звучит из фона:** не возвращайся к `WMPlayer`/`MediaPlayer` — используй именно `mciSendString`.
