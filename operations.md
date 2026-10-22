# Operations

## Health checks

```bash
docker compose ps
docker compose logs --tail=200 quasar-server processor
docker compose exec postgres pg_isready -U quasar -d quasar
curl http://localhost:8090/health
curl http://localhost:31121/devices
```

For services without a health endpoint, check logs, TCP listeners, and functional APIs. `push-service` has no HTTP listener; monitor its MQTT connection logs.

## Updates

```bash
git pull
docker compose build
docker compose up -d
```

Do not remove volumes without a backup. Use `pg_dump` for PostgreSQL and back up the `speaker-data` volume for enrollment profiles.

## Logs and diagnostics

```bash
docker compose logs -f SERVICE_NAME
docker compose config
```

The first ASR and speaker-identifier startup can be slow because models are downloaded from external registries. Ensure outbound HTTPS is allowed.

## Security

Use unique secrets, do not expose `31121` or `9010` without authentication, replace the private CA in `https-proxy`, restrict MQTT ACLs, and never place `.env` in GitHub Pages.
