# CleanSweep

Portable-деинсталятор и клинер для Windows на Free Pascal / Lazarus —
один exe (~1.1 МБ после UPX), без .NET. Эвристики поиска остатков собраны по мотивам
Geek Uninstaller / Uninstall Tool / BCUninstaller / DeepPurge; движок
очистки портирован с FluentCleaner (winapp2-формат).

## Возможности

- **Деинсталляция**: штатный деинсталлятор программы (с тихими флагами
  для MSI/Inno/NSIS/InstallShield/WiX) → скан остатков → ревью → удаление.
- **Поиск остатков без удаления**: ПКМ по программе → «Найти остатки
  (без удаления)…» — предпросмотр хвостов до деинсталляции. Глубина —
  Меню → «Глубина поиска остатков» (Быстрый/Стандартный/Глубокий).
- **Live-exe guard**: папка с живым `.exe` (не инсталляторская заглушка)
  — review-only и не удаляется даже при ручной отметке.
- **Клинер**: Analyze → ревью → Clean по базам Winapp2/Winapp3/Default/
  CCleaner + свои правила в `Custom\*.ini`; пресеты, security-вкладка
  «Пароли и сессии» (никогда не предотмечена).
- **Свои правила**: визуальный редактор, «Проверить…» (реальный Analyze),
  «В Custom» — клонирование записей базы для изучения/правки.
- **Куки**: точечная чистка по keep-list внутри SQLite (winsqlite3.dll),
  менеджер «Куки…» — файлы БД не удаляются.
- **AppX-деблоатер**: пункт «AppX» в главном меню — удаление UWP/MSIX
  пакетов по базе Winappx.ini (только установленные).
- **Память**: пункт «Память» — разовая очистка ОЗУ (рабочие наборы,
  standby-листы, файловый кэш) с отчётом-тостом «до → после МБ»
  в стиле MemCleaner.
- **Сеть**: пункт «Сеть» — 11 операций сетевого ремонта (flush
  DNS/NetBIOS/ARP, сброс Winsock и стека TCP/IP, route -f,
  release/renew, рестарт адаптеров) с цветной колонкой «Последствия»
  и журналом выполнения.
- **Прочее**: обновление баз по сети, спец-очистка (корзина, мёртвые
  ярлыки), локализация RU/BE/UK/EN, `CleanSweep.ini`/`CleanSweep.log`
  рядом с exe.

## Ядро

- `cs.registry.pas` — сканер установленных программ: HKLM (64-бит view),
  WOW6432Node, HKCU, HKU SID'ы. Дедупликация, поля Uninstall-ключей.
- `cs.leftovers.pas` — умная посточистка. TScanContext — единый снапшот
  на операцию; token-aware matching; boundary-safe IsUnder; dedup;
  confidence-скоринг (Safe/Moderate/Risky).
  - реестр: software-ключи (2 уровня), App Paths, Run/RunOnce,
    StartupApproved, сервисы, COM CLSID, firewall rules, MuiCache,
    BAM, RecentApps, PATH env;
  - файлы: InstallLocation, Program Files(x86), ProgramData,
    AppData Roaming/Local/LocalLow, VirtualStore, TEMP,
    Start Menu, Desktop, Quick Launch, SendTo, Prefetch, JumpLists;
  - scheduled tasks через schtasks;
  - проверка запущенных процессов перед удалением.
- `cs.safety.pas` — SafetyGuard: защищённые каталоги/файлы/ветки реестра,
  сервисы, задачи, firewall-префиксы, проверка reparse point.
- `cs.signatures.pas` — загрузка `data\leftover-signatures.json`.
  JSON вшит в exe ресурсом `LEFTOVERSIG` (RCDATA, как в DocSigner);
  внешний файл, если лежит рядом, имеет приоритет.
- `cs.uninstall.pas` — запуск uninstaller'а (CreateProcessW + Job Object
  + таймаут), `ScanOnly` для предпросмотра без деинсталляции, удаление
  остатков (файлы в корзину SHFileOperationW, реестр RegDeleteTreeW с
  WOW64-view); `lcRisky` не удаляется никогда.
- `gui/*.lfm` + `*.pas` — формы в ресурсах: главная, карточка программы
  (даблклик), модальные диалоги — редактируются в дизайнере.

## Сборка

```
build.bat
```

Требуется Lazarus/FPC в `C:\lazarus` (FPC 3.2.2 i386 — бинарь 32-битный,
читает 64-битный реестр через `KEY_WOW64_64KEY`).

Манифест требует `requireAdministrator`.

## Параметры командной строки

- `-c` — открыть клинер
- `-e` (`-net`) — форму сброса сетевых настроек
- `-m` (`-mem`) — разовую очистку памяти с тостом-отчётом у трея
- `-auto` — автономная очистка в трее: чистит то, что отмечено в
  клинере (записи с `Warning=` не выполняются); для Планировщика задач
- `-h` — «О программе»; F1 — справка из любой формы

Каждый параметр запускает только свой инструмент (окно деинсталлятора
не создаётся): закрыли форму — программа завершилась.

## Документация

- `DEVELOPER.md` — руководство разработчика (архитектура, сборка, грабли)
- `docs\user-guide.md` — справка пользователя
- `docs\custom-rules.md` — синтаксис Custom\*.ini
- `docs\decisions.md` — лог решений и изменений (по датам)
- `docs\port-plan.md` — план порта FluentCleaner
- `docs\cleaner-comparison.md` — сравнение с FluentCleaner/CCleaner

## Не реализовано (backlog)

- Мониторинг установок в реальном времени (Install Tracker)
- Steam, Chocolatey
- Portable-сканер произвольных папок
- Резервные копии .reg перед удалением
