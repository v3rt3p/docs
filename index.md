---
layout: default
title: Razvalisa Documentation
---

# Razvalisa

Документация по развертыванию набора сервисов голосового ассистента.

## Развертывание

1. Установите Docker Engine и Docker Compose v2.
2. Скопируйте `.env.example` в `.env` в корне репозитория.
3. Заполните `OPENAI_API_KEY`, `MQTT_URL`, `POSTGRES_PASSWORD` и `GLAGOL_JWT_KEY`.
4. Выполните `docker compose build && docker compose up -d`.
5. Проверьте `GET http://localhost:8090/health` и `GET http://localhost:31121/devices`.

Полная архитектура описана в [архитектуре](architecture.md), а команды эксплуатации — в [операционном руководстве](operations.md). Compose-файл находится в корне репозитория и запускается с локального сервера.

## Навигация

- [Полный walkthrough](deployment.md)
- [Архитектура](architecture.md)
- [Эксплуатация](operations.md)
- [Сервисы](services.md)

## Компоненты

- [alice-tts](services/alice-tts.md) — синтез речи.
- [function-state-server](services/function-state-server.md) — состояния и функции через MQTT.
- [https-proxy](services/https-proxy.md) — TLS reverse proxy.
- [nlp-service](services/nlp-service.md) — детерминированный parser.
- [processor](services/processor.md) — LLM processor.
- [push-service](services/push-service.md) — MQTT push bridge.
- [quasar-server](services/quasar-server.md) — Quasar backend.
- [quasistreaming-gigam](services/quasistreaming-gigam.md) — ASR.
- [speaker-identifier](services/speaker-identifier.md) — speaker embeddings.

## Важно

GitHub Pages предназначен только для документации. Сервисы, Docker Compose, секреты и модели запускаются на вашем сервере, а не внутри GitHub Pages.
