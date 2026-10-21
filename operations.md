# Эксплуатация

## Проверка состояния

```bash
docker compose ps
docker compose logs --tail=200 quasar-server processor
docker compose exec postgres pg_isready -U quasar -d quasar
curl http://localhost:8090/health
curl http://localhost:31121/devices
```

У сервисов без health endpoint проверяйте логи, TCP listener и рабочий API. `push-service` не имеет HTTP listener; его состояние проверяется по MQTT connection logs.

## Обновление

```bash
git pull
docker compose build
docker compose up -d
```

Volumes не удаляйте без резервной копии. Для базы сделайте `pg_dump`; для speaker profiles сохраните volume `speaker-data`.

## Логи и диагностика

```bash
docker compose logs -f SERVICE

docker compose config
```

Первый запуск ASR и speaker identifier может быть долгим: модели скачиваются из внешних registry. Убедитесь, что контейнерам разрешен исходящий HTTPS.

## Безопасность

Задайте уникальные секреты, не публикуйте `31121` и `9010` без авторизации, замените приватный CA в `https-proxy`, ограничьте MQTT ACL и не размещайте `.env` в GitHub Pages.
