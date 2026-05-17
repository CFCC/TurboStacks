# MDM (`mdm`) — Fleet

Docker Compose stack for [Fleet](https://fleetdm.com/) with TLS terminated by Traefik (same pattern as [`logging/`](../logging/)).

## Prerequisites

- Docker `container-dmz` network exists (see [`router/docker-compose.yaml`](../router/docker-compose.yaml)).
- Traefik is running with the `namecheap` certificate resolver configured.
- DNS `mdm.apps.campcomputer.com` points at your Traefik entrypoint.

Fleet listens on plain HTTP (`FLEET_SERVER_TLS=false`) on port **1337** inside the stack; HTTPS is only at the edge via Traefik.

**Images:** Fleet and MySQL publish multi-arch images. If anything fails on non-AMD64 hosts, validate image support for your platform before deploying.

## Setup

1. Copy the template secrets file:

   `cp env.example /opt/CampFitch/secrets/mdm.env`

2. Edit `/opt/CampFitch/secrets/mdm.env`: set strong MySQL passwords, align `MYSQL_PASSWORD` / `MYSQL_USER` / `MYSQL_DATABASE` with `FLEET_MYSQL_*`, and replace `FLEET_SERVER_PRIVATE_KEY` (e.g. `openssl rand -base64 32`).

Only `MYSQL_*` variables are consumed by the `mysql` container; everything else applies to Fleet. Keeping both MySQL and Fleet DB settings in one file matches this stack’s Compose layout.

3. Optional: Behind Traefik, if client IPs matter or Websockets misbehave, see the commented `FLEET_SERVER_TRUSTED_PROXIES` and `FLEET_SERVER_WEBSOCKETS_ALLOW_UNSAFE_ORIGIN` lines in [`env.example`](env.example).

## Start

From this directory:

`docker compose up -d`

## Image updates

The Fleet container image is pinned in [`docker-compose.yaml`](docker-compose.yaml) (see `fleetdm/fleet:<tag>`). To upgrade, pick a stable tag from [Fleet releases](https://github.com/fleetdm/fleet/releases), adjust the Compose file (and optionally align `env`/upgrade notes), then `docker compose pull && docker compose up -d`.

## Apple MDM (APNs, SCEP, ABM)

Configure MDM **after** Fleet is running (wizard/UI and env), following Fleet’s [MDM setup](https://fleetdm.com/docs/using-fleet/mdm-setup).

## Reference

- [Deploy Fleet with Docker Compose](https://fleetdm.com/guides/deploy-fleet-on-docker-compose) — healthchecks, volumes, migrations, operational details.
