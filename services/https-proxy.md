# https-proxy

Nginx reverse proxy для доменов из `domains/*.txt`. При старте генерирует сертификаты и конфигурацию.

## Docker Compose

```bash
docker compose build https-proxy
YANDEX_UPSTREAM_URL=https://proxy.example docker compose up -d https-proxy
```

Standalone: `docker build -t v3rt3p-https-proxy . && docker run --rm -p 80:80 -p 443:443 -e YANDEX_UPSTREAM_URL=https://proxy.example v3rt3p-https-proxy`.

## Настройка

Первая строка каждого списка в `domains/` — upstream, остальные строки — проксируемые домены. Для `domains/yandex.txt` override задается как `YANDEX_UPSTREAM_URL`; аналогично `VK_UPSTREAM_URL` и `SBER_UPSTREAM_URL`. `NGINX_ROOT` меняет корень путей генератора.

`./generate.sh` генерирует `generated/`; затем Nginx слушает `80` и `443` и поддерживает WebSocket upgrade. `/health` отсутствует.

## Безопасность

В репозитории присутствует приватный CA-ключ: перед публикацией замените его, не отдавайте клиентам `ca.key`, а сгенерированные ключи храните вне Git.
