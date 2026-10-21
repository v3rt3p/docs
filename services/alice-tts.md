# alice-tts

HTTP speech synthesis service using the Yandex Alice UniProxy WebSocket API.

## Docker Compose

From the deployment repository:

```bash
docker compose build alice-tts
docker compose up -d alice-tts
curl -X POST http://localhost:9150/v1/audio/speech -H 'content-type: application/json' -d '{"input":"Hello"}' --output answer.ogg
```

Standalone: `docker build -t alice-tts . && docker run --rm -p 9150:9150 -e PORT=9150 alice-tts`.

## Configuration

- `PORT` - HTTP port, default `9150`.
- `CACHE_SIZE` - configured cache size.
- `DUMP_PATH` - optional directory for OGG files and metadata.
- `UNIWS_URL` - UniProxy WebSocket URL, default `wss://uniproxy.alice.yandex.net/uni.ws`.

Endpoints are `POST /v1/audio/speech` and `POST /audio/speech`. The request body is `{ "input": "..." }` and the response is `audio/ogg`. There is no formal `/health` endpoint.

Use a persistent volume for `DUMP_PATH`, expose the service through a protected reverse proxy, and verify outbound connectivity to UniProxy.
