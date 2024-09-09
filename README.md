[![Continuous Integration][continuous-integration-badge]][continuous-integration-link]
[![OpenSSF Scorecard][scorecard-badge]][scorecard-link]

<header align="center">
    <h1 align="center">Docker User Mirror</h1>
    <p align="center">Mirror host user ownership in rootful Docker containers.</p>
</header>

## Why?
The goal of this project is to unify bind mount permissions, regardless of the container engine context. Whether you're using rootful Docker, rootless Docker, or Podman, this project will configure your host environment and container to ensure bind mount items are owned by the host user, not root.

The primary focus of this project is to improve the developer experience of working with containerized toolchains. Rather than requiring a rootless context, you can incorporate Docker User Mirror to become context agnostic!

## Getting Started
1. Append `Dockerfile.user-mirror` to your Dockerfile.
2. Copy `entrypoint` to the directory of your Dockerfile.
3. Copy `user-mirror` to the directory of your `docker-compose.yml`.

## Usage
The `user-mirror` script combined with the `entrypoint` script will mirror the host user inside the container. This alongside creating bind mount objects before the Docker daemon eliminates ownership mismatches when bind mounting on Linux.

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
[continuous-integration-badge]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/ci.yml/badge.svg?branch=main
[continuous-integration-link]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/ci.yml
[reddit-expanded-community-filter-userscript]: https://github.com/AJGranowski/reddit-expanded-community-filter-userscript
[scorecard-badge]: https://api.securityscorecards.dev/projects/github.com/AJGranowski/docker-user-mirror/badge
[scorecard-link]: https://securityscorecards.dev/viewer/?uri=github.com/AJGranowski/docker-user-mirror
