# Инструкция по установке S-UI v1.4.1

Эта инструкция описывает установку архивной/кастомной сборки **S-UI v1.4.1**.

В сборке есть:

```text
sui
libcronet.so
s-ui-custom-1.4.1-docker-image.tar
BUILD_INFO.txt
```

Рекомендуемый способ установки: **Docker-образ**.

## 1. Требования

Рекомендуемая ОС:

```text
Ubuntu 22.04 / 24.04
Debian 12
```

Нужные пакеты:

```bash
sudo apt update
```

```bash
sudo apt install -y docker.io curl
```

Запустить и включить Docker:

```bash
sudo systemctl enable --now docker
```

Проверить Docker:

```bash
sudo docker --version
```

## 2. Установка через Docker-образ

Перейди в папку, где лежат файлы релиза:

```bash
cd /path/to/releases
```

Загрузи Docker-образ:

```bash
sudo docker load -i s-ui-custom-1.4.1-docker-image.tar
```

Проверь, что образ появился:

```bash
sudo docker images | grep s-ui-custom
```

Запусти S-UI:

```bash
sudo docker run -d --name s-ui --restart unless-stopped --network host -v s-ui-data:/app/db s-ui-custom:1.4.1
```

Проверь статус контейнера:

```bash
sudo docker ps
```

Проверь логи:

```bash
sudo docker logs -f s-ui
```

## 3. Открытие панели

Стандартный адрес панели:

```text
http://SERVER_IP:2095/app/
```

Стандартный адрес подписок:

```text
http://SERVER_IP:2096/sub/
```

Если тестируешь внутри WSL, узнай IP WSL:

```bash
hostname -I
```

Потом открой:

```text
http://WSL_IP:2095/app/
```

## 4. Firewall

Разрешить порт панели:

```bash
sudo ufw allow 2095/tcp
```

Разрешить порт подписок:

```bash
sudo ufw allow 2096/tcp
```

Перезагрузить UFW:

```bash
sudo ufw reload
```

Проверить статус:

```bash
sudo ufw status
```

На публичном сервере лучше не оставлять админку открытой для всего интернета. Безопаснее ограничить доступ по IP, VPN или reverse proxy.

## 5. Остановка, запуск, перезапуск

Остановить S-UI:

```bash
sudo docker stop s-ui
```

Запустить S-UI:

```bash
sudo docker start s-ui
```

Перезапустить S-UI:

```bash
sudo docker restart s-ui
```

Посмотреть логи:

```bash
sudo docker logs -f s-ui
```

## 6. Резервная копия данных

Данные контейнера хранятся в Docker volume:

```text
s-ui-data
```

Создать backup:

```bash
sudo docker run --rm -v s-ui-data:/data -v "$PWD":/backup alpine tar czf /backup/s-ui-data-backup.tar.gz -C /data .
```

Восстановить backup:

```bash
sudo docker stop s-ui
```

```bash
sudo docker run --rm -v s-ui-data:/data -v "$PWD":/backup alpine sh -c "rm -rf /data/* && tar xzf /backup/s-ui-data-backup.tar.gz -C /data"
```

```bash
sudo docker start s-ui
```

## 7. Обновление контейнера из нового образа

Остановить и удалить старый контейнер:

```bash
sudo docker stop s-ui
```

```bash
sudo docker rm s-ui
```

Загрузить новый образ:

```bash
sudo docker load -i s-ui-custom-1.4.1-docker-image.tar
```

Запустить заново с тем же volume данных:

```bash
sudo docker run -d --name s-ui --restart unless-stopped --network host -v s-ui-data:/app/db s-ui-custom:1.4.1
```

## 8. Дополнительно: запуск standalone-бинарника

Рекомендуется использовать Docker. Этот вариант нужен только если понимаешь, что делаешь.

Создать папку установки:

```bash
sudo mkdir -p /opt/s-ui
```

Скопировать файлы:

```bash
sudo cp sui /opt/s-ui/
```

```bash
sudo cp libcronet.so /opt/s-ui/
```

Сделать бинарник исполняемым:

```bash
sudo chmod +x /opt/s-ui/sui
```

Запустить вручную:

```bash
cd /opt/s-ui
```

```bash
sudo LD_LIBRARY_PATH=/opt/s-ui ./sui
```

## 9. Диагностика

Проверить, слушает ли панель порты:

```bash
sudo ss -lntp | grep -E '2095|2096'
```

Посмотреть логи контейнера:

```bash
sudo docker logs --tail=100 s-ui
```

Проверить ответ панели локально:

```bash
curl -I http://127.0.0.1:2095/app/
```

Ответ `307 Temporary Redirect` на `/app/login` означает, что панель работает корректно.

## 10. Примечания

Это архивная/кастомная сборка S-UI.

Исходный код frontend был восстановлен из:

```text
ClashConnectRules/s-ui-frontend
commit f65f58e
```

Используйте на свой риск.
