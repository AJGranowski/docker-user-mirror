<header align="center">
    <h1 align="center">Docker User Mirror</h1>
    <p align="center">Mirror the host user inside containers.</p>
</header>

# ⚠️ THIS PROJECT IS A WORK IN PROGRESS ⚠️

## Getting Started
1. Append `Dockerfile.user-mirror` to your Dockerfile.
2. Copy `entrypoint` to the directory of your Dockerfile.
3. Copy `docker-user-mirror-args` and `podman-user-mirror-args` to the directory of your `docker-compose.yml`.

## Usage
The `docker-user-mirror-args` and `podman-user-mirror-args` scripts combined with the `entrypoint` script mirror the host user inside the container. This eliminates permission mismatches when bind mounting on Linux.

```shell
docker compose build
docker compose run $(./docker-user-mirror-args) --rm {service}
```

```shell
podman compose build
podman compose run $(./podman-user-mirror-args) --rm {service}
```