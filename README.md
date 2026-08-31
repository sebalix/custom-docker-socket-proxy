# custom-docker-socket-proxy

Custom Docker Socket Proxy exposing the Docker Engine API on `127.0.0.1:2375` with a restricted set of allowed endpoints.

It is especially useful for limiting what an AI agent (or any untrusted client) can do with Docker, by restricting it to a whitelisted set of endpoints.

## Prerequisites

- Docker and the Docker Compose plugin installed.
- The `docker-socket-proxy:local` image built locally (built from https://github.com/sebalix/docker-socket-proxy/tree/imp-methods-access-granularity).

## Manual usage

```bash
docker compose up -d
docker compose down
```

## How to use it

Point any Docker client at the proxy:

```bash
export DOCKER_HOST=tcp://localhost:2375
docker ps
```

## Start automatically at login (systemd user service)

Create `~/.config/systemd/user/custom-docker-socket-proxy.service`:

```ini
[Unit]
Description=Custom Docker Socket Proxy (compose)
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/path/to/custom-docker-socket-proxy
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0

[Install]
WantedBy=default.target
```

Enable and start it:

```bash
systemctl --user daemon-reload
systemctl --user enable --now custom-docker-socket-proxy.service
```

Verify:

```bash
systemctl --user status custom-docker-socket-proxy.service
docker compose ps
```

Notes:

- `WantedBy=default.target` triggers the service at login.
- `After=docker.service` waits for the Docker daemon to be up first.
- `RemainAfterExit=yes` combined with `ExecStop` runs `docker compose down` cleanly on logout/shutdown.
