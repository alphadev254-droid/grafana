# Grafana Observability Stack

This stack runs Grafana, Loki, Prometheus, Tempo, and Grafana Alloy in Docker while your apps keep running directly on the host.

## Start

```powershell
Copy-Item .env.example .env
docker compose up -d
```

Open Grafana locally:

```text
http://localhost:3000
```

Default login comes from `.env`.

## Safe Remote Access

On a server, keep ports bound to `127.0.0.1` and use SSH tunneling:

```bash
ssh -L 3000:localhost:3000 user@your-server
```

Then open:

```text
http://localhost:3000
```

## Logs

Alloy reads log files from:

```text
./logs/**/*.log
```

For host apps, write or symlink app logs into this folder, for example:

```bash
ln -s /var/www/app/storage/logs/laravel.log ./logs/laravel.log
ln -s /var/log/icims-backend/app.log ./logs/icims-backend.log
```

Logs go to Loki and are available in Grafana Explore.

## Metrics

Prometheus is internal to Docker by default, so it does not bind host port `9090`. Grafana reaches it through the Docker network at `http://prometheus:9090`.

Built-in metrics:

```text
node-exporter:9100   # VPS CPU, RAM, disk, network
cadvisor:8080        # Docker container CPU/RAM
```

Prometheus scrapes:

```text
host.docker.internal:5000/metrics
host.docker.internal:8000/metrics
```

The default backend scrape job is `icims-backend` on `host.docker.internal:5000/metrics`. Update `prometheus/prometheus.yml` if your backend uses a different port.

For Node.js, expose `/metrics` with `prom-client`.

## Traces

Apps can send OTLP traces to Alloy:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
```

Alloy forwards traces to Tempo. Grafana reads Tempo as a datasource.

Tempo runs as root in this Compose file so it can write to the Docker-managed local trace volume. Keep Tempo bound to `127.0.0.1` unless you put it behind a private network.

## Useful Commands

```powershell
docker compose ps
docker compose logs -f alloy
docker compose logs -f grafana
docker compose down
```

To delete stored monitoring data:

```powershell
docker compose down -v
```
