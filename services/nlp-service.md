# nlp-service

Deterministic natural-language parser for common assistant requests. It does not use MQTT.

## Docker Compose

```bash
docker compose build nlp-service
docker compose up -d nlp-service
curl http://localhost:8090/health
curl -X POST http://localhost:8090/parse -H 'content-type: application/json' -d '{"text":"turn on the light","functions":{}}'
```

Standalone: `docker build -t nlp-service . && docker run --rm -p 8090:8090 -e PORT=8090 nlp-service`.

## Environment and API

- `PORT` (default `8090`)
- `TIMEZONE` (default `Europe/Moscow`)
- `GET /health` returns `{ "status": "ok" }`.
- `POST /parse` accepts `{ "text": string, "functions": object }`; JSON body limit is 64 KiB.

Run `pnpm test && pnpm typecheck` before deployment.
