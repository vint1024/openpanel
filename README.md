# OpenPanel — image builds

This fork builds the Docker images of [OpenPanel](https://github.com/Openpanel-dev/openpanel)
(AGPL-3.0) from upstream release tags, **without any changes to the code**. Upstream publishes
its v3 images to supporters only; the source is public, so we build it ourselves.

| Image | Source |
|---|---|
| `ghcr.io/vint1024/openpanel-api:<version>` (also the worker, with `ROLE=worker`) | tag `v<version>`, `apps/api/Dockerfile` |
| `ghcr.io/vint1024/openpanel-dashboard:<version>` | tag `v<version>`, `apps/start/Dockerfile` |

`linux/amd64` and `linux/arm64`. The source of every image is the tag `v<version>` in this
repository, identical to upstream. This branch (`vint/build`) holds only the build workflow,
see [`.github/workflows/build-images.yml`](.github/workflows/build-images.yml).

## Building a new version

```bash
git fetch --depth=1 https://github.com/Openpanel-dev/openpanel.git refs/tags/v3.20.0:refs/tags/v3.20.0
git push git@github.com:vint1024/openpanel.git v3.20.0
gh workflow run build-images.yml -R vint1024/openpanel -f version=3.20.0 -f node_version=24.22.2
```

Only release tags are copied here. `main` is left as it was at fork time and is never synced,
so the upstream workflows (which live on `main`) never run in this fork.

## Runtimes

The Dockerfiles take Node and Bun as build arguments (`ARG NODE_VERSION`, `ARG BUN_VERSION`).
We build with the newest patch release of the line the tag pins — the same major for Node,
the same major.minor for Bun — so a fix released after the tag is picked up without touching
the code. Inputs left empty keep the tag's own versions. The runtimes of every image are in its
`org.opencontainers.image.description` label.

| Version | Node (dashboard) | Bun (api, worker) |
|---|---|---|
| 3.20.0 | 24.22.2 (tag: 24.22.1) | 1.4.4 (= tag) |

Node 26 became LTS on 2026-10-28; upstream requires Node ≥ 24.22.1 and builds on 24, so we stay
on the 24 line until upstream moves.

## Services we run it with

The images need PostgreSQL, ClickHouse, a Redis-compatible store and a Kafka-compatible broker.
Upstream's `self-hosting/docker-compose.template.yml` (at v3.20.0) pins some of them to versions
that no longer get updates, so we run newer ones (checked 2026-11-02):

| Service | Upstream compose | We run | Why |
|---|---|---|---|
| Redpanda | `v25.1.4` | `v26.2.6` | 25.1 is past end of life since 2026-04-07 ([docs](https://docs.redpanda.com/25.1/upgrade/rolling-upgrade/)); Redpanda upgrades one feature release at a time, so a fresh install starts on the current one |
| ClickHouse | `25.10.2.65` | `26.8.36.3` (LTS) | 25.* gets no security fixes ([SECURITY.md](https://github.com/ClickHouse/ClickHouse/blob/master/SECURITY.md)); an LTS is supported for a year |
| PostgreSQL | `14` | `18.6` | 14 reaches end of life on 2026-11-12 ([versioning policy](https://www.postgresql.org/support/versioning/)); 19 is still a release candidate |
| Redis | `redis 7.2.5` | Valkey `9.1.3` | Redis-compatible; the current Valkey release |

Tested with a copy of a 2.3.0 install: PostgreSQL restored from a dump into 18, ClickHouse and
Valkey upgraded in place, then 3.20.0 migrations, login, ingestion through Redpanda and the
realtime view. Settings that matter on a small host: Valkey must have room for one session and
one cached profile per active device (about 3 KB per device seen in the last hour, with
`noeviction`), and the worker needs more memory than in 2.x (it peaked at ~750 MB at ~700 events/s).
