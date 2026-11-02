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
gh workflow run build-images.yml -R vint1024/openpanel -f version=3.20.0
```

The upstream workflows of this fork are disabled.
