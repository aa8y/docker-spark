# Docker Spark

[![CI](https://github.com/aa8y/docker-spark/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/aa8y/docker-spark/actions/workflows/ci.yml)

[Apache Spark](http://spark.apache.org) is a framework for doing distributed Big Data processing. This project contains files to build a Docker image for Spark. It is a fork of [semantive/spark](https://github.com/Semantive/docker-spark) but has been modified to use Alpine as base to make the final image smaller. It can be used in a standalone cluster or with the accompanying `docker-compose.yml` as a base for more complex recipes.

## Simple example

To run `SparkPi`, run the image with Docker:

```
docker run --rm -it -p 4040:4040 aa8y/spark bin/run-example SparkPi 10
```

## Cluster example [docker-compose]

To create a simple standalone cluster with [docker-compose](http://docs.docker.com/compose) use:

```
docker-compose up
```

The SparkUI will be running at `http://${YOUR_DOCKER_HOST}:8080` with one worker listed and Spark jobs may be submitted using master `spark://${YOUR_DOCKER_HOST}:7077`. To connect via spark-shell with cluster use:

```
spark-shell --master spark://localhost:7077
```

## Tags

Only the latest patch of each supported Spark minor line is published.
`latest` tracks the newest stable; `3` is an alias for the newest 3.x.

| Tag      | Spark version | Hadoop  | JDK |
|----------|---------------|---------|-----|
| `2.4.8`  | 2.4.8         | 2.7.7   | 8   |
| `2.4`    | 2.4.8         | 2.7.7   | 8   |
| `3.5.8`  | 3.5.8         | 3.3.6   | 17  |
| `3.5`    | 3.5.8         | 3.3.6   | 17  |
| `3`      | 3.5.8         | 3.3.6   | 17  |
| `latest` | 3.5.8         | 3.3.6   | 17  |

Spark 2.4 is pinned to JDK 8 because the 2.x line never ran on JDK 11+.
Spark 3.5 ships JDK 8/11/17 support; we pair it with JDK 17 (Amazon
Corretto via `aa8y/core:jdk17`) as the most forward-leaning supported
option. The pre-2.4 tags (`1.6.x`, `2.0.x`–`2.3.x`) and the
source-built `edge-*` tags have been retired — Apache EOL'd those
lines and the edge build no longer applies cleanly to current Spark.
See `manifest.yml` for the full matrix.

### Building / Pushing / Tagging docker images

The project uses [Dave](https://github.com/aa8y/dave) to build, test, and push images, driven by GitHub Actions on every push to `master`. To build additional tags, fork the repository, edit `manifest.yml`, and let the workflow in `.github/workflows/ci.yml` take it from there.

### Testing

Image tests are defined as [container-structure-test][cst] configs under
`test/config/` — a shared `common.yaml` plus a version-banner check and a
SparkPi end-to-end run. The configs apply to every `stable` tag (declared
once in `manifest.yml` under `structureTest:`) and run natively via
`dave structure-test`:

```sh
brew install container-structure-test     # one-time

dave build
dave structure-test
```

CI runs the same commands; see `.github/workflows/ci.yml`.

[cst]: https://github.com/GoogleContainerTools/container-structure-test

### Dockerfile

* [Stable](https://github.com/aa8y/docker-spark/blob/master/stable/Dockerfile)

## License

Apache Licence
