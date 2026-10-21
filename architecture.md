# Архитектура

Основной поток запроса: Yandex Station подключается к `quasar-server` по WebSocket; Quasar передает запросы в `processor`; processor использует LLM, NLP и remote state/function servers. Для аудио Quasar обращается к GigaAM ASR и TTS backend. `function-state-server` работает с MQTT. `push-service` получает MQTT events и отправляет push в API Quasar.

## Внешние зависимости

- PostgreSQL для `quasar-server`.
- MQTT broker для state/function и push событий.
- OpenAI-compatible LLM для `processor`.
- Hugging Face для первой загрузки GigaAM и Qwen speaker model.
- NVIDIA runtime только для CUDA ASR.

## Сетевые имена Compose

Внутри сети Docker используются `postgres`, `processor`, `function-state-server`, `nlp-service`, `quasistreaming-gigam`, `alice-tts` и `quasar-server`. Не используйте `localhost` для обращения одного контейнера к другому.
