# quasar-server

Backend-эмулятор Yandex Station Quasar: WebSocket listener для станций и отдельный HTTP API. Требует PostgreSQL, processor и STT/TTS backend.

## Docker Compose

```bash
docker compose build quasar-server
docker compose up -d postgres processor quasar-server
# API: http://localhost:31121, основной listener: http://localhost:31120
curl http://localhost:31121/devices
```

Standalone: `docker build -t quasar-server . && docker run --rm -p 31120:31120 -p 31121:31121 --env-file .env quasar-server`.

## Переменные

- `PORT` (default `31115`, project `.env` обычно `31120`)
- `API_PORT` (default `31116`, project `.env` обычно `31121`)
- `POSTGRES_URL`, например `postgres://quasar:quasar@postgres:5432/quasar`
- `PROCESSOR_BASIC_URL`, например `http://processor:8080`
- `STT_GIGAAM_URL`, WebSocket URL ASR
- `TTS_OPENAI_BASE_URL`, `TTS_OPENAI_API_KEY`, `TTS_OPENAI_MODEL`, `TTS_OPENAI_VOICE`, `TTS_OPENAI_SPEED`
- `AUDIO_METADATA_URLS`
- `GLAGOL_JWT_KEY` — задайте стабильный секрет

Явного `/health` нет; `GET /devices` — только техническая проверка API. Перед production проверьте PostgreSQL и реальные STT/TTS endpoints. `synchronize: true` в TypeORM требует отдельной оценки для production.
