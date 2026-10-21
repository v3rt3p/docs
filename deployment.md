# Полный walkthrough развёртывания

## 1. Что входит в систему

Razvalisa состоит из нескольких процессов, которые запускаются в одной Docker Compose-сети:

1. `quasar-server` принимает подключения Yandex Station и предоставляет API для push-событий.
2. `processor` принимает WebSocket-сессии от Quasar, собирает контекст и вызывает OpenAI-compatible LLM.
3. `nlp-service` быстро разбирает типовые команды без обращения к LLM.
4. `function-state-server` предоставляет processor состояния и функции, а их источником/транспортом служит MQTT.
5. `quasistreaming-gigam` распознаёт входной звук по WebSocket.
6. `alice-tts` превращает текст в OGG через Yandex UniProxy.
7. `push-service` переносит MQTT-события в HTTP API Quasar.
8. `speaker-identifier` хранит и сравнивает voice embeddings.
9. `postgres` хранит зарегистрированные станции и данные Quasar.
10. `https-proxy` является опциональным внешним TLS reverse proxy.

GitHub Pages содержит только эту документацию. Docker-контейнеры должны работать на вашем сервере, VPS или локальном Docker host.

## 2. Требования

- Docker Engine 24 или новее.
- Docker Compose v2 (`docker compose version`).
- Git.
- Для GPU ASR: NVIDIA driver, NVIDIA Container Toolkit и совместимая CUDA-система.
- Исходящий HTTPS-доступ для загрузки npm/pip-пакетов и моделей GigaAM/Qwen.
- Доступный MQTT broker.
- Доступный OpenAI-compatible LLM endpoint.

Проверьте установку:

```bash
docker --version
docker compose version
git --version
```

## 3. Получение исходников

Общий `docker-compose.yml` предполагает, что репозитории сервисов находятся рядом с каталогом `docs` в одном parent-каталоге:

```bash
mkdir razvalisa && cd razvalisa
git clone https://github.com/v3rt3p/docs.git docs
git clone https://github.com/v3rt3p/alice-tts.git alice-tts
git clone https://github.com/v3rt3p/function-state-server.git function-state-server
git clone https://github.com/v3rt3p/https-proxy.git https-proxy
git clone https://github.com/v3rt3p/nlp-service.git nlp-service
git clone https://github.com/v3rt3p/processor.git processor
git clone https://github.com/v3rt3p/push-service.git push-service
git clone https://github.com/v3rt3p/quasar-server.git quasar-server
git clone https://github.com/v3rt3p/quasistreaming-gigam.git quasistreaming-gigam
git clone https://github.com/v3rt3p/speaker-identifier.git speaker-identifier
```

Команды ниже выполняются из `razvalisa/docs`, где лежит Compose-файл.

## 4. Секреты и конфигурация

```bash
cp .env.example .env
chmod 600 .env
```

Откройте `.env` и обязательно замените значения:

```dotenv
POSTGRES_PASSWORD=<long-random-password>
OPENAI_API_KEY=<llm-api-key>
MQTT_URL=mqtts://<user>:<password>@<broker>:8883
GLAGOL_JWT_KEY=<long-random-secret>
```

Не добавляйте `.env` в Git. Не публикуйте MQTT credentials, LLM keys, `GLAGOL_JWT_KEY`, `ca.key` или enrollment profiles.

### State/function configuration

Перед запуском проверьте `function-state-server/config.yaml`. Пустой файл запускается, но не объявляет функций и состояний. За основу можно взять `config.example.yaml`, затем указать реальные MQTT topics и внешние HTTP endpoints.

### Quasar configuration

В Compose уже заданы внутренние имена Docker:

- PostgreSQL: `postgres:5432`.
- Processor: `processor:8080`.
- ASR: `quasistreaming-gigam:8080`.
- TTS: `alice-tts:9150`.
- Quasar API: `quasar-server:31121`.

Внутри контейнера нельзя использовать `localhost` для другого сервиса.

## 5. Первый запуск

Проверьте итоговую конфигурацию без запуска:

```bash
docker compose --env-file .env config
```

Соберите образы:

```bash
docker compose --env-file .env build
```

Запустите систему:

```bash
docker compose --env-file .env up -d
```

Посмотрите состояние:

```bash
docker compose ps
docker compose logs -f quasar-server processor
```

Первый запуск ASR и speaker identifier может занять значительное время: модели загружаются из внешних источников. Их cache сохраняется в named volumes.

## 6. Проверка после запуска

NLP имеет формальный health endpoint:

```bash
curl -fsS http://localhost:8090/health
```

Проверка Quasar API:

```bash
curl -i http://localhost:31121/devices
```

Проверка processor выполняется через WebSocket endpoint `/process`; отдельного HTTP `/health` у него нет. Проверяйте его логи и подключение Quasar.

Проверка ASR выполняется тестовым клиентом из репозитория:

```bash
cd ../quasistreaming-gigam
python client.py input.wav ws://localhost:8080
```

WAV должен быть mono или stereo PCM с частотой 16 kHz. Внутри Compose ASR доступен по адресу `ws://quasistreaming-gigam:8080`, а наружу по умолчанию не публикуется.

TTS проверяется через Quasar или напрямую:

```bash
curl -X POST http://localhost:9150/v1/audio/speech \
  -H 'content-type: application/json' \
  -d '{"input":"Привет"}' \
  --output answer.ogg
```

Чтобы выполнить последнюю команду с хоста, временно опубликуйте порт `9150` через override-файл либо используйте `docker compose exec`.

## 7. GPU ASR

CPU — безопасный вариант по умолчанию. Для CUDA:

1. Установите NVIDIA Container Toolkit.
2. Проверьте `docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi`.
3. Соберите `quasistreaming-gigam` с `Dockerfile.cuda`.
4. Задайте `GIGAAM_DEVICE=cuda`.
5. Добавьте GPU reservation из upstream `quasistreaming-gigam/docker-compose.yaml`.
6. Убедитесь в логах, что указано `device=cuda`.

Если модель переключилась на CPU, проверьте driver, toolkit, PyTorch wheel и поддерживаемую архитектуру GPU.

## 8. HTTPS proxy

Proxy запускается только при необходимости:

```bash
docker compose --env-file .env --profile proxy up -d https-proxy
```

Порты:

- HTTP: `80`.
- HTTPS: `443`.

Перед production замените CA-материалы в `https-proxy/certs/`. Приватный ключ нельзя публиковать в GitHub. Настройки upstream задаются через `YANDEX_UPSTREAM_URL`, `VK_UPSTREAM_URL` и `SBER_UPSTREAM_URL`.

## 9. Данные и backup

Compose создаёт volumes:

- `postgres-data` — PostgreSQL.
- `gigaam-cache` — checkpoint GigaAM.
- `speaker-models` — cache модели Qwen.
- `speaker-data` — enrollment profiles.

Перед обновлением сделайте backup PostgreSQL и каталога profiles. Не выполняйте `docker compose down -v`, если данные должны сохраниться.

## 10. Обновление и остановка

```bash
git -C ../alice-tts pull
git -C ../processor pull
git -C ../quasar-server pull
docker compose --env-file .env build
docker compose --env-file .env up -d
```

Остановить контейнеры без удаления volumes:

```bash
docker compose down
```

Логи конкретного сервиса:

```bash
docker compose logs --tail=200 -f SERVICE_NAME
```

## 11. Типовые проблемы

- **`function-state-server` пустой:** заполните `config.yaml`, иначе processor не увидит функции.
- **`processor` не вызывает LLM:** проверьте `OPENAI_API_KEY`, `OPENAI_BASE_URL` и `OPENAI_MODEL`.
- **Quasar не видит processor:** внутри Compose должен использоваться `http://processor:8080`, а не `localhost`.
- **ASR долго стартует:** дождитесь загрузки модели и проверьте volume `gigaam-cache`.
- **MQTT не подключается:** проверьте URL, credentials, CA certificate и сетевой доступ.
- **PostgreSQL недоступен:** дождитесь `service_healthy`, проверьте `POSTGRES_URL` и `docker compose logs postgres`.
- **Push-события пропадают:** `push-service` не гарантирует доставку и не имеет persistent queue; проверяйте MQTT и API logs.

## 12. Production checklist

- [ ] Все секреты заменены и отсутствуют в Git.
- [ ] MQTT broker использует ACL и TLS.
- [ ] API ports закрыты firewall или защищены reverse proxy.
- [ ] PostgreSQL регулярно резервируется.
- [ ] `GLAGOL_JWT_KEY` стабилен между рестартами.
- [ ] Модель ASR заранее прогрета.
- [ ] Speaker verification откалибрована на target/impostor recordings.
- [ ] Проверена стратегия миграций PostgreSQL; `synchronize: true` не оставлен без оценки.
- [ ] Настроены restart policy, мониторинг логов и дискового пространства.
