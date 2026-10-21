# quasistreaming-gigam

WebSocket ASR на GigaAM `v3_e2e_rnnt` и TEN VAD. Принимает mono PCM16, 16 kHz; endpointing возвращает JSON с `text` и `end_of_utt`.

## Compose

CUDA-вариант уже описан в `docker-compose.yaml`:

```bash
docker compose build
docker compose up -d
docker compose logs -f
```

Нужны NVIDIA driver, NVIDIA Container Toolkit и GPU 0. CPU-вариант:

```bash
docker build -f Dockerfile.cpu -t quasistreaming-gigam:cpu .
docker run --rm -p 8080:8080 -v gigaam-cache:/app/.cache/gigaam quasistreaming-gigam:cpu
```

## Конфигурация

`PORT`, `GIGAAM_DEVICE=auto|cpu|cuda`, `GIGAAM_DOWNLOAD_ROOT`, `GIGAAM_MODEL_NAME`, `VAD_THRESHOLD`, `VAD_MIN_SILENCE_DURATION`, `INPUT_GAIN`, `ASR_STREAMING_MODE`. При первом старте нужен интернет и persistent volume модели. Проверка: `python client.py input.wav ws://localhost:8080`.
