[![CI/CD][cicd-badge]][cicd-link]
[![OpenSSF Scorecard][scorecard-badge]][scorecard-link]

<header align="center">
    <h1 align="center">Docker User Mirror</h1>
    <p align="center">Mirror host user ownership in and out of rootful Docker containers.</p>
</header>

## Why?
The goal of this project is to unify bind mount permissions, regardless of the container engine context. Whether you're using rootful Docker, rootless Docker, or Podman, this project will configure your host environment and container to ensure bind mount items are owned by the host user, not root.

The primary focus of this project is to improve the developer experience of working with containerized toolchains. Rather than requiring a rootless context, you can incorporate Docker User Mirror to become context agnostic!

#### Example
Lets say you want to mount a volume inside a bind mount.
```shell
-v ./:/app -v /app/volume
```

Here's what would happen with rootful Docker on Linux systems:
```shell
(IMAGE_ID="debian:latest"; \
docker run --rm -u "$(id -u):$(id -g)" -v ./:/app -v /app/volume -w /app "$IMAGE_ID" sh -c \
'id && find volume -printf "%m %u:%g %f (container)\n"' && \
docker image rm "$IMAGE_ID" >/dev/null) && \
find volume -printf "%m %u:%g %f (host)\n" && \
rm -rf volume
```
```
uid=1000 gid=1000 groups=1000
755 root:root volume (container) ⇐😭
755 root:root volume (host)      ⇐😭
```

Even though we used the `-u, --user` option to mirror the host user inside the container, `volume` still ends up being owned by `root`! What a pain!

Thankfully, that's where this project steps in to help:
```shell
(IMAGE_ID="$(docker build -q image/)"; \
./user-mirror docker run --rm -v ./:/app -v /app/volume -w /app "$IMAGE_ID" sh -c \
'id && find volume -printf "%m %u:%g %f (container)\n"' && \
docker image rm "$IMAGE_ID" >/dev/null) && \
find volume -printf "%m %u:%g %f (host)\n" && \
rm -rf volume
```
```
uid=1000(user) gid=1000(user) groups=1000(user)
755 user:user volume (container) ⇐🥳
755 user:user volume (host)      ⇐🥳
```

It even works with rootless Docker and Podman!
```
uid=0(root) gid=0(root) groups=0(root)
755 root:root volume (container)
755 user:user volume (host)
```

## Getting Started
1. Copy `entrypoint` to the directory of your Dockerfile.
2. Copy `user-mirror` to the directory of your `docker-compose.yml`.
3. Append `Dockerfile.user-mirror` to your Dockerfile.
4. Merge `docker-compose.yml` with your existing `docker-compose.yml` (`cap_add`, `cap_drop`, `entrypoint`, `environment`).

## Usage
The `user-mirror` script combined with the `entrypoint` script mirror the host user inside the container, and `chown` objects created by Docker. This alongside preemptive creation of host bind mount objects before the Docker daemon eliminates ownership mismatches.

```shell
docker compose build
./user-mirror docker compose run --rm {service}
```

```shell
podman compose build
./user-mirror podman compose run --rm {service}
```

## Development
Run `ci` to run the test scripts in `test/` with the images in `images/` using both Docker Compose and Podman Compose (if installed).

----

<h2 align="center">Partner Projects</h2>

* [Reddit Expanded Community Filter][reddit-expanded-community-filter-userscript]

[best-practices-badge]: https://www.bestpractices.dev/
[best-practices-link]: https://www.bestpractices.dev/
[cicd-badge]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/cicd.yml/badge.svg?branch=main
[cicd-link]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/cicd.yml
[reddit-expanded-community-filter-userscript]: https://github.com/AJGranowski/reddit-expanded-community-filter-userscript
[scorecard-badge]: https://api.securityscorecards.dev/projects/github.com/AJGranowski/docker-user-mirror/badge
[scorecard-link]: https://securityscorecards.dev/viewer/?uri=github.com/AJGranowski/docker-user-mirror
