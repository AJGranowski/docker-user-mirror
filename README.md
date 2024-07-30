<header align="center">
    <h3 align="center">⚠️ THIS PROJECT IS A WORK IN PROGRESS ⚠️</h3>
</header>

[![Continuous Integration][continuous-integration-badge]][continuous-integration-link]
[![OpenSSF Scorecard][scorecard-badge]][scorecard-link]

<header align="center">
    <h1 align="center">Docker User Mirror</h1>
    <p align="center">Mirror the host user inside containers.</p>
</header>

## Getting Started
1. Append `Dockerfile.user-mirror` to your Dockerfile.
2. Copy `entrypoint` to the directory of your Dockerfile.
3. Copy `user-mirror-args` to the directory of your `docker-compose.yml`.

## Usage
The `user-mirror-args` script combined with the `entrypoint` script mirror the host user inside the container. This eliminates permission mismatches when bind mounting on Linux.

```shell
docker compose build
docker compose run $(./user-mirror-args --docker) --rm {service}
```

```shell
podman compose build
podman compose run $(./user-mirror-args --podman) --rm {service}
```

## Development
Run `ci` to run the test scripts in `test/` with the images in `images/` using both Docker Compose and Podman Compose (if installed).

[best-practices-badge]: https://www.bestpractices.dev/
[best-practices-link]: https://www.bestpractices.dev/
[continuous-integration-badge]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/ci.yml/badge.svg?branch=main
[continuous-integration-link]: https://github.com/AJGranowski/docker-user-mirror/actions/workflows/ci.yml
[scorecard-badge]: https://api.securityscorecards.dev/projects/github.com/AJGranowski/docker-user-mirror/badge
[scorecard-link]: https://securityscorecards.dev/viewer/?uri=github.com/AJGranowski/docker-user-mirror
