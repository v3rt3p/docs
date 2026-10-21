# push-service

MQTT-to-HTTP bridge: получает события MQTT и отправляет их в API `quasar-server`. HTTP-порта у сервиса нет.

## Docker Compose

```bash
docker compose build push-service
docker compose up -d push-service
docker compose logs -f push-service
```

Standalone: `docker build -t push-service . && docker run --rm --env-file .env push-service`.

## Переменные

- `API_URLS` — comma-separated API URLs; в общем Compose используйте `http://quasar-server:31121`.
- `MQTT_URL`
- `MQTT_CA_CERTIFICATE_PATH` (default `ca-cert.pem`)
- `MQTT_TOPIC_PREFIX` (default `bus/services/alice`)

Подписываются topics `event`, `event/raw` и варианты с device id. `/health` отсутствует. События не имеют гарантированной доставки: нет retry/очереди, поэтому мониторьте MQTT и логи.
