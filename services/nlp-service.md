# nlp-service

Детерминированный parser запросов ассистента. MQTT не использует.

## Docker Compose

```bash
docker compose build nlp-service
docker compose up -d nlp-service
curl http://localhost:8090/health
curl -X POST http://localhost:8090/parse -H 'content-type: application/json' -d '{"text":"включи свет","functions":{}}'
```

Standalone: `docker build -t nlp-service . && docker run --rm -p 8090:8090 -e PORT=8090 nlp-service`.

## Переменные и API

- `PORT` (default `8090`)
- `TIMEZONE` (default `Europe/Moscow`)
- `GET /health` -> `{ "status": "ok" }`
- `POST /parse` принимает `{ "text": string, "functions": object }`; лимит JSON body — 64 KiB.

Проверка перед деплоем: `pnpm test && pnpm typecheck`.
