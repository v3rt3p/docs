# push-service

MQTT-to-HTTP bridge that receives events from MQTT and sends them to the `quasar-server` API. It does not expose an HTTP port.

## Docker Compose

```bash
docker compose build push-service
docker compose up -d push-service
docker compose logs -f push-service
```

Standalone: `docker build -t push-service . && docker run --rm --env-file .env push-service`.

## Environment

- `API_URLS` - comma-separated API URLs; use `http://quasar-server:31121` in the shared Compose network.
- `MQTT_URL`
- `MQTT_CA_CERTIFICATE_PATH` (default `ca-cert.pem`)
- `MQTT_TOPIC_PREFIX` (default `bus/services/alice`)

The service subscribes to event topics and device-specific variants. There is no `/health` endpoint. Delivery is not guaranteed: there is no retry queue or durable event store, so monitor MQTT and API logs.
