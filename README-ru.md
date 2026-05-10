# S-UI Backup

Личный backup / survival-архив панели **S-UI**.

Оригинальный upstream-репозиторий `alireza0/s-ui` стал недоступен для меня и начал отдавать `404` на сам репозиторий, releases, raw install script и профиль автора. Этот репозиторий сохранён как личная резервная копия заведомо рабочей установки S-UI.

Это **не официальный репозиторий S-UI**.

## Репозиторий

```text
https://github.com/FireTIA/S_UI_Backup
```

## Статус

Архивированная рабочая установка:

```text
Панель: S-UI
Core: на базе sing-box
Дата бэкапа: 2026-05-11
Оригинальный путь установки: /usr/local/s-ui
Имя бинарника: sui
Команда управления: s-ui
Systemd service: s-ui.service
```

## Структура репозитория

Ожидаемая структура:

```text
S_UI_Backup/
├── README.md
├── README-ru.md
├── checksums.md5
├── checksums.hash
├── checksums.sha1
├── checksums.sha256
├── checksums.sha512
└── root/
    └── s-ui-survival/
        ├── file-info.txt
        ├── s-ui-command
        ├── s-ui.service
        ├── s-ui.sh
        ├── sui.bin
        ├── s-ui-systemd-cat.txt
        ├── s-ui-status.txt
        ├── tree-lite.txt
        └── s-ui-full-2026-05-11/
            └── usr/
                └── local/
                    └── s-ui/
                        ├── db/
                        ├── s-ui.service
                        ├── s-ui.sh
                        └── sui
```

Папка `db/` намеренно оставлена без реальной базы данных.

## Важное предупреждение по безопасности

**Не загружайте настоящий `s-ui.db` в публичный GitHub-репозиторий.**

Реальная база S-UI может содержать:

- настройки админ-панели;
- пользователей;
- UUID;
- Reality keys;
- данные подписок;
- домены;
- порты;
- лимиты трафика;
- другую чувствительную конфигурацию сервера.

В публичном бэкапе не должно быть этих файлов:

```text
s-ui.db
s-ui.db-wal
s-ui.db-shm
*.sqlite
*.sqlite3
```

Рекомендуемый `.gitignore`:

```gitignore
*.db
*.db-wal
*.db-shm
*.sqlite
*.sqlite3
*.log
```

## Установка / восстановление на новую VPS

Целевая система: Ubuntu/Debian с systemd.

Установить нужные пакеты:

```bash
sudo apt update
sudo apt install -y git sqlite3 ca-certificates
```

Склонировать репозиторий:

```bash
git clone https://github.com/FireTIA/S_UI_Backup.git
cd S_UI_Backup/root/s-ui-survival
```

Остановить старый сервис S-UI, если он уже был установлен:

```bash
sudo systemctl stop s-ui 2>/dev/null || true
```

Создать директории установки:

```bash
sudo mkdir -p /usr/local/s-ui/db
```

Скопировать файлы S-UI:

```bash
sudo cp -f sui.bin /usr/local/s-ui/sui
sudo cp -f s-ui.sh /usr/local/s-ui/s-ui.sh
sudo cp -f s-ui.service /usr/local/s-ui/s-ui.service
```

Выдать права на запуск:

```bash
sudo chmod +x /usr/local/s-ui/sui
sudo chmod +x /usr/local/s-ui/s-ui.sh
```

Установить systemd service:

```bash
sudo cp -f /usr/local/s-ui/s-ui.service /etc/systemd/system/s-ui.service
```

Установить команду управления `s-ui`:

```bash
sudo cp -f s-ui-command /usr/bin/s-ui
sudo chmod +x /usr/bin/s-ui
```

Перезагрузить systemd и запустить S-UI:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now s-ui
```

Проверить статус сервиса:

```bash
sudo systemctl status s-ui --no-pager
```

Открыть меню управления S-UI:

```bash
sudo s-ui
```

## Альтернативное восстановление из архивированной папки

Если удобнее восстановить установку из сохранённого дерева файлов, а не из отдельных файлов:

```bash
cd S_UI_Backup/root/s-ui-survival
```

Скопировать сохранённое дерево установки:

```bash
sudo mkdir -p /usr/local/s-ui
sudo cp -a s-ui-full-2026-05-11/usr/local/s-ui/. /usr/local/s-ui/
```

Выдать права на запуск:

```bash
sudo chmod +x /usr/local/s-ui/sui
sudo chmod +x /usr/local/s-ui/s-ui.sh
```

Установить service и команду управления:

```bash
sudo cp -f /usr/local/s-ui/s-ui.service /etc/systemd/system/s-ui.service
sudo cp -f s-ui-command /usr/bin/s-ui
sudo chmod +x /usr/bin/s-ui
```

Запустить S-UI:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now s-ui
sudo s-ui
```

## Восстановление приватного бэкапа базы

Делайте это только если у вас есть **свой приватный** бэкап `s-ui.db`.

**Не используйте базу данных из непроверенного источника.**

Остановить S-UI:

```bash
sudo systemctl stop s-ui
```

Удалить старые файлы базы:

```bash
sudo rm -f /usr/local/s-ui/db/s-ui.db
sudo rm -f /usr/local/s-ui/db/s-ui.db-wal
sudo rm -f /usr/local/s-ui/db/s-ui.db-shm
```

Скопировать приватную базу:

```bash
sudo cp -f s-ui.db /usr/local/s-ui/db/s-ui.db
```

Исправить владельца и права:

```bash
sudo chown root:root /usr/local/s-ui/db/s-ui.db
sudo chmod 600 /usr/local/s-ui/db/s-ui.db
```

Проверить целостность SQLite-базы:

```bash
sudo sqlite3 /usr/local/s-ui/db/s-ui.db "PRAGMA integrity_check;"
```

Запустить S-UI:

```bash
sudo systemctl start s-ui
```

Проверить статус:

```bash
sudo systemctl status s-ui --no-pager
```

## Проверка файлов

Из корня репозитория:

```bash
cd S_UI_Backup
```

Проверить SHA256-суммы:

```bash
sha256sum -c checksums.sha256
```

Также в репозитории есть другие checksum-файлы:

```text
checksums.md5
checksums.sha1
checksums.sha512
checksums.hash
```

Если ваша утилита проверки использует другой формат путей, можно вручную сравнить хэши важных файлов:

```bash
sha256sum root/s-ui-survival/sui.bin
sha256sum root/s-ui-survival/s-ui.sh
sha256sum root/s-ui-survival/s-ui.service
sha256sum root/s-ui-survival/s-ui-command
```

## Логи

Посмотреть логи сервиса S-UI:

```bash
sudo journalctl -u s-ui -e --no-pager
```

Смотреть логи в реальном времени:

```bash
sudo journalctl -u s-ui -f
```

## Полезные команды

Открыть меню управления:

```bash
sudo s-ui
```

Перезапустить S-UI:

```bash
sudo systemctl restart s-ui
```

Остановить S-UI:

```bash
sudo systemctl stop s-ui
```

Запустить S-UI:

```bash
sudo systemctl start s-ui
```

Проверить состояние сервиса:

```bash
sudo systemctl status s-ui --no-pager
```

## Заметки

Не используйте встроенную функцию обновления, если вы точно не знаете, куда указывает источник обновлений.

Оригинальный upstream исчез для меня, поэтому функция обновления может не работать или сломать установку.

Рекомендуемый подход:

- использовать этот репозиторий как архив;
- сначала проверять восстановление на отдельной VPS;
- не перезаписывать рабочую production-установку без приватного бэкапа;
- хранить реальные бэкапы базы только приватно;
- держать хотя бы одну офлайн-копию оригинального полного бэкапа.

## Дисклеймер

Этот репозиторий является только архивной копией рабочей установки S-UI.

Он не является официальным, не поддерживается оригинальным автором и не даёт никаких гарантий.

Используйте на свой риск.
