[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Snowplow Docker Setup](Snowplow-Docker-Setup)

## Overview

Snowplow provides official Docker images for the following projects:

- [Scala Stream Collector](Scala-Stream-Collector)
- [Stream Enrich](Stream-Enrich)
- [Elasticsearch Loader](Elasticsearch-Loader)
- [S3 Loader](S3-Loader)
- [Iglu Server][iglu-server]

## Getting started

First off, you need to have [Docker][docker] installed. For more information check out the
[installation guide][installation-guide].

## Installing the Docker images

You can choose to either:

1. Pull the Docker images from our registry, _or:_
2. Build them yourself

### Pulling the Docker images from our registry

Our Docker images are hosted on the [`snowplow-docker-registry.bintray.io`][registry] docker
registry. As a result, you can pull them directly:

```bash
# Scala Stream Collector image
docker pull snowplow-docker-registry.bintray.io/snowplow/scala-stream-collector:0.13.0

# Stream Enrich image
docker pull snowplow-docker-registry.bintray.io/snowplow/stream-enrich:0.16.0

# Elasticsearch Loader image
docker pull snowplow-docker-registry.bintray.io/snowplow/elasticsearch-loader:0.10.1

# S3 Loader image
docker pull snowplow-docker-registry.bintray.io/snowplow/s3-loader:0.6.0

# Iglu Server image
docker pull snowplow-docker-registry.bintray.io/snowplow/iglu-server:0.3.0
```

### Building the Docker images

Alternatively, you can build each image from the Dockerfiles in the repository.

To do so, clone the snowplow-docker repo:

```bash
git clone https://github.com/snowplow/snowplow-docker.git
```

You can now build each image:

```bash
# All images are based on the base image
docker build -t snowplow/base:0.1.0 base

# Scala Stream Collector image
docker build -t snowplow/scala-stream-collector:0.13.0 scala-stream-collector/0.13.0

# Stream Enrich image
docker build -t snowplow/stream-enrich:0.16.0 stream-enrich/0.16.0

# Elasticsearch Loader image
docker build -t snowplow/elasticsearch-loader:0.10.0 elasticsearch-loader/0.10.1

# S3 Loader image
docker build -t snowplow/s3-loader:0.6.0 s3-loader/0.6.0

# Iglu Server image
docker build -t snowplow/iglu-server:0.3.0 iglu-server/0.3.0
```

## Using the images

### Configuration

For each project, please refer to the corresponding setup guide to configure it appropriately:

- [Scala Stream Collector setup guide](Configure-the-Scala-Stream-Collector)
- [Stream Enrich setup guide](Configure-Stream-Enrich)
- [Elasticsearch Loader setup guide](Elasticsearch-Loader-Setup)
- [S3 Loader setup guide](Snowplow-S3-Loader-Setup)
- [Iglu Server setup guide][iglu-server-setup]

### Execution

Once you have your configuration files ready to go for each project, containers can be launched
with the following:

```bash
# Scala Stream Collector container
docker run \
  -v $PWD/scala-stream-collector-config:/snowplow/config \
  snowplow/scala-stream-collector:0.13.0 \
  --config /snowplow/config/config.hocon

# Stream Enrich
docker run \
  -v $PWD/stream-enrich-config:/snowplow/config \
  snowplow/stream-enrich:0.16.0 \
  --config /snowplow/config/config.hocon \
  --resolver file:/snowplow/config/resolver.json \
  --enrichments file:/snowplow/config/enrichments/ \
  --force-ip-lookups-download

# Elasticsearch Loader
docker run \
  -v $PWD/elasticsearch-loader-config:/snowplow/config \
  snowplow/elasticsearch-loader:0.10.1 \
  --config /snowplow/config/config.hocon

# S3 Loader
docker run \
  -v $PWD/s3-loader-config:/snowplow/config \
  snowplow/s3-loader:0.6.0 \
  --config /snowplow/config/config.hocon

# Iglu Server
docker run \
-v ${PWD}/iglu-server-config:/snowplow/config \
snowplow/iglu-server:0.3.0 \ # if you have built the image
# snowplow-docker-registry.bintray.io/snowplow/iglu-server:0.3.0 if you have pulled the image
--config /snowplow/config/application.conf
```

[docker]: https://www.docker.com
[installation-guide]: https://docs.docker.com/engine/installation/
[registry]: https://bintray.com/snowplow/registry
[iglu-server]: https://github.com/snowplow/iglu/wiki/Iglu-server
[iglu-server-setup]: https://github.com/snowplow/iglu/wiki/Iglu-server-setup

