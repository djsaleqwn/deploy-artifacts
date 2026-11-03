# Deploy Artifacts

Binary artifacts needed to deploy in restricted-network environments (offline Docker Engine bundles, the Compose binary, common container image tarballs, etc.).

Git repositories are a poor place for binaries of tens or hundreds of megabytes (100 MiB per-file hard limit), so all artifacts are published as **release assets**, tagged by component + platform, so installer scripts can fetch exactly what they need.

## Why this repository exists

Many production hosts (especially overseas or air-gapped ones) cannot reach Docker Hub, GitHub directly, or the OS package repositories, yet the installation needs those large files. Publishing them as public release assets lets the installer:

- fetch them on demand when the host has internet access, with no manual copying;
- verify sha256 before use, so a replaced or corrupted file is never installed;
- fall back to other sources (online install or a local offline bundle) when the download fails.

## Contents

| Tag | Contents | Platforms |
|---|---|---|
| `docker-el7-v2` | Docker Engine offline bundle (self-contained RPM set, system dependencies included) | CentOS 7 / RHEL 7, x86_64 |
| `docker-el9-v2` | Docker Engine offline bundle (differential RPM closure: only Docker components plus the packages they are missing) | RHEL 9 family (Rocky 9 / AlmaLinux 9 / RHEL 9 / CentOS Stream 9), x86_64 |
| `docker-ubuntu-2204-v1` | Docker Engine offline bundle (self-contained deb set, system dependencies included) | Ubuntu 22.04 LTS, x86_64 |
| `docker-ubuntu-2404-v1` | Docker Engine offline bundle (self-contained deb set, system dependencies included) | Ubuntu 24.04 LTS, x86_64 |
| `compose-v5.1.3` | Docker Compose standalone static binary | linux x86_64 |
| `images-el7-v1` | Common container image tarballs (mysql / nginx / openresty / temurin-8-jre / coturn / redis, one file per image) | linux amd64 (distribution independent) |

Each release note states the purpose, the sha256, and how to use the asset.

Note: `docker-el9-v1` (older) is a **full closure** built on CentOS Stream 9 that includes base packages such as systemd and glibc; installing it on Rocky / Alma / RHEL 9.4 is rejected by dnf as replacing protected packages. **Use `docker-el9-v2` instead.**

Note: the tarballs under `images-*` are `docker save` output and are independent of the host distribution; any amd64 Linux Docker can run `docker load -i <file>`. The file names are a fixed contract (the installer looks them up by name) - do not rename them.

## Download

Direct URL template:

```text
https://github.com/djsaleqwn/deploy-artifacts/releases/download/<tag>/<asset>
```

Direct downloads of large files from GitHub are often slow or return zero bytes on some networks; a public proxy prefix can be used instead (either one):

```text
https://ghfast.top/https://github.com/djsaleqwn/deploy-artifacts/releases/download/<tag>/<asset>
https://gh-proxy.com/https://github.com/djsaleqwn/deploy-artifacts/releases/download/<tag>/<asset>
```

Example (verify the result before unpacking):

```bash
url=https://github.com/djsaleqwn/deploy-artifacts/releases/download/compose-v5.1.3/docker-compose-linux-x86_64-5.1.3
curl -fL --connect-timeout 5 --speed-limit 102400 --speed-time 30 -o /tmp/docker-compose "$url"
sha256sum /tmp/docker-compose
chmod +x /tmp/docker-compose
```

## Naming rules

- **Tag**: `<component>-<platform>-v<number>`, e.g. `docker-el7-v2`, `compose-v5.1.3`, `images-el7-v1`.
- **Asset file name**: keep the original name expected by the consumer (e.g. `docker-offline.tar.gz`) - do not rename it, or the installer will not find it.
- When publishing updated artifacts, **create a new tag**; never overwrite assets on an existing tag - hosts already deployed may depend on the old version.

## Verification

The correct sha256 of every asset is also written in its release note. Verify before use:

```bash
sha256sum <file>   # macOS: shasum -a 256 <file>
```

## License and disclaimer

This repository only distributes artifacts. Every artifact remains the property of its upstream project and keeps its upstream license (for example Docker Engine is Apache-2.0). No warranty is provided; confirm compatibility with your target environment yourself.

This repository contains **no** source code, business applications, keys, credentials, or customer data.

