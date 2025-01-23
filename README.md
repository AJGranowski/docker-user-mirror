[![Total Downloads][downloads-badge]](#)
[![CI/CD][cicd-badge]][cicd-link]
[![OpenSSF Scorecard][scorecard-badge]][scorecard-link]
[![OpenSSF Best Practices][best-practices-badge]][best-practices-link]

<header align="center">
    <h1 align="center">Docker User Mirror</h1>
    <p align="center">Mirror host user ownership in and out of rootful Docker containers.</p>
</header>

## How It Works
This projects uses a pair of scripts to mirror the host user in a container:

```shell
docker compose build
./user-mirror docker compose run --rm {{service}}
```

1. `entrypoint` (build time): Install `setpriv` or `gosu`.
2. `user-mirror`: Create mount source items on the host before Docker does (so they're owned by the current host user rather than root).
3. `user-mirror`: Inject environment variables into the Docker/Podman command and run it.
4. `entrypoint` (run time): Create a mirrored host user in the container from the injected environment variables.
5. `entrypoint` (run time): `chown` mount destination items in the container to the mirrored user.
6. `entrypoint` (run time): Step-down from root to the mirrored user to execute the command.

## Getting Started
1. Copy the `entrypoint` script to the directory of your Dockerfile.
2. Copy the `user-mirror` script to the root of your project.
3. Append `Dockerfile.user-mirror` to your project's Dockerfile.
4. Add `cap_add`, `cap_drop`, and `environment` from `compose.yml` to your project's compose file.
5. Prepend your project's compose entrypoint with `[/entrypoint, --,` ...

## Why?
Lets say you want to mount a volume inside a bind mount.
```
-v ./:/app -v /app/volume
      ↑            ↖ volume mount within the exiting bind mount (/app)
  bind mount
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

Also compatible with rootless Docker and Podman!
```
uid=0(root) gid=0(root) groups=0(root)
755 root:root volume (container)
755 user:user volume (host)
```

The big benefit to using this project over [other workarounds](https://github.com/moby/moby/issues/2259) is that it's all automatic. The file ownership fixes are all inferred directly from your command or compose specification.

## Development
Run `ci` to run the test scripts in `test/` with the images in `images/` using both Docker Compose and Podman Compose (if installed).

----

<h2 align="center">Partner Projects</h2>

* [Reddit Expanded Community Filter][reddit-expanded-community-filter-userscript]

[best-practices-badge]: https://www.bestpractices.dev/projects/9502/badge
[best-practices-link]: https://www.bestpractices.dev/projects/9502
[cicd-badge]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/cicd.yml/badge.svg?branch=main
[cicd-link]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/cicd.yml
[downloads-badge]: https://img.shields.io/github/downloads/AJGranowski/docker-user-mirror/user-mirror?logo=github&label=Total%20downloads&labelColor=30373d&color=4078c0
[reddit-expanded-community-filter-userscript]: https://github.com/AJGranowski/reddit-expanded-community-filter-userscript
[scorecard-badge]: https://api.securityscorecards.dev/projects/github.com/AJGranowski/docker-user-mirror/badge
[scorecard-link]: https://securityscorecards.dev/viewer/?uri=github.com/AJGranowski/docker-user-mirror
