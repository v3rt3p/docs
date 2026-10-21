# alice-tts

HTTP-сервис синтеза речи через Yandex Alice UniProxy.

## Docker Compose

Из корня проекта выполните:

```bash
docker compose build alice-tts
docker compose up -d alice-tts
curl -X POST http://localhost:9150/v1/audio/speech -H 'content-type: application/json' -d '{"input":"Привет"}' --output answer.ogg
```

Без общего Compose: `docker build -t alice-tts . && docker run --rm -p 9150:9150 -e PORT=9150 alice-tts`.

## Настройка

- `PORT` — HTTP-порт, по умолчанию `9150`.
- `CACHE_SIZE` — заявленный размер LRU-кэша.
- `DUMP_PATH` — необязательная директория для OGG и metadata.
- `UNIWS_URL` — WebSocket UniProxy; по умолчанию `wss://uniproxy.alice.yandex.net/uni.ws`.

Endpoints: `POST /v1/audio/speech` и `POST /audio/speech`, тело `{ "input": "..." }`, ответ `audio/ogg`. Формального `/health` нет.

## Production

Используйте persistent volume для `DUMP_PATH`, ограничьте доступ к порту reverse proxy и проверьте доступ контейнера к UniProxy. Dockerfile не содержит `EXPOSE`, поэтому порт публикуется явно.
