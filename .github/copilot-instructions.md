# Copilot / AI Agent Instructions for plg

This project is a small, self-hosted observability stack composed with Docker Compose. The guidance below is focused, actionable, and specific to this repo so an AI coding agent can be immediately productive.

- **Big picture:** The repository runs Prometheus, Loki, Promtail, and Grafana via `docker-compose.yml`. Promtail collects logs and pushes to Loki. Prometheus scrapes exporters and Caddy metrics. Grafana loads dashboards from `grafana/provisioning`.

- **Key services & files:**
  - Prometheus: `prometheus/prometheus.yml` (scrape targets, global scrape interval)
  - Loki: `loki/loki-config.yml` (storage, wal, retention)
  - Promtail: `promtail/promtail-config.yml` (log scrapes; note `__path__` entries like `/var/log/caddy/*.log`)
  - Grafana provisioning: `grafana/provisioning/` and dashboards under `grafana/provisioning/dashboards` and `grafana/dashboards`
  - Compose entry: `docker-compose.yml` (volumes, port mappings, env values)

- **How pieces communicate:**
  - Promtail -> Loki: via `http://loki:3100/loki/api/v1/push` (see `promtail/promtail-config.yml`).
  - Prometheus scrapes services by Docker hostnames declared in `prometheus.yml` (e.g., `prometheus:9090`, `cadvisor:8080`).
  - Grafana reads dashboards/datasources from `grafana/provisioning` on container startup.

- **Local development / common commands:**
  - Start services: `docker compose up -d` (root directory). Use `docker compose logs -f <service>` to tail logs.
  - Inspect Prometheus UI: http://localhost:9090 ; Grafana: http://localhost:3000 (admin password from `docker-compose.yml` env `GF_SECURITY_ADMIN_PASSWORD` = `admin`).
  - To test log flow: write a line into a file matched by Promtail (e.g., `echo "test" >> /var/log/caddy/test.log`) then query Loki from Grafana Explore.

- **Project-specific conventions / gotchas:**
  - Loki uses host bind-mounted volumes for data and WAL (`device: "./loki"` and `./wal`) — avoid destructive volume changes without backups.
  - `host.docker.internal` is used in `prometheus.yml` for scraping Caddy metrics on macOS; keep that when editing scrape targets on macOS.
  - Grafana dashboards are provisioned; prefer editing JSON in `grafana/provisioning/dashboards/` and reloading Grafana rather than editing via UI for reproducibility.

- **When changing configs:**
  - Update the config file in the repo (e.g., `prometheus/prometheus.yml`, `loki/loki-config.yml`) and restart the affected service: `docker compose up -d <service>`.
  - For structural changes to `docker-compose.yml` (ports, volumes), call out the impact to persisted volumes — these are declared under `volumes:` and some use `driver_opts` with binds.

- **Testing & CI:**
  - There is no test harness or CI configured in the repo. Changes should be validated locally by bringing up the compose stack and verifying service UIs and logs.

- **Editing and PR guidance for AI agents:**
  - Keep commits small and focused: one config change per PR with a short description (what changed and why). Example commit message: `prometheus: add new scrape job for myservice`
  - Do not alter the data directories or volume `device` paths unless explicitly instructed by a human.

- **Examples to cite in edits or suggestions:**
  - If adding a new log source, mirror the `promtail` job structure (see `promtail/promtail-config.yml`, `job_name: caddy`).
  - If adding a new Grafana dashboard, place JSON under `grafana/provisioning/dashboards/` and update `grafana/provisioning/dashboards/dashboards.yml`.

If anything here is unclear or you want more detail (e.g., preferred commit conventions or a CI workflow), tell me which area to expand and I will iterate.
