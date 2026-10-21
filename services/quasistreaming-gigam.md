# quasistreaming-gigam

WebSocket ASR service using GigaAM `v3_e2e_rnnt` and TEN VAD. It accepts mono PCM16 audio at 16 kHz and emits JSON containing `text` and `end_of_utt`.

## Compose

The CUDA deployment is described in `docker-compose.yaml`:

```bash
docker compose build
docker compose up -d
docker compose logs -f
```

It requires an NVIDIA driver, NVIDIA Container Toolkit, and GPU 0. CPU deployment:

```bash
docker build -f Dockerfile.cpu -t quasistreaming-gigam:cpu .
docker run --rm -p 8080:8080 -v gigaam-cache:/app/.cache/gigaam quasistreaming-gigam:cpu
```

## Configuration

Use `PORT`, `GIGAAM_DEVICE=auto|cpu|cuda`, `GIGAAM_DOWNLOAD_ROOT`, `GIGAAM_MODEL_NAME`, `VAD_THRESHOLD`, `VAD_MIN_SILENCE_DURATION`, `INPUT_GAIN`, and `ASR_STREAMING_MODE`. The first start requires internet access and a persistent model volume. Test with `python client.py input.wav ws://localhost:8080`.
