[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) > Snowplow Docker

Snowplow provides official Docker images for the following projects:

- [Scala Stream Collector](Scala-Stream-Collector)
- [Stream Enrich](Stream-Enrich)
- [Elasticsearch Loader](Elasticsearch-Loader)
- [S3 Loader](S3-Loader)
- [Iglu Server][iglu-server]

All the images are based on [the base image][base-image] which leverages
[the Java 8 Alpine image][alpine-image].

Each component runs under [dumb-init][dumb-init] which handles reaping zombie processes
and forwards signals on to all processes running in the container. This image also uses
[su-exec][su-exec], as a sudo replacement, to run the component as the non-root `snowplow` user.

Each container exposes the `/snowplow/config` volume to store the component's configuration. If this
folder is bind mounted then ownership will be changed to the `snowplow` user.

The `-XX:+UnlockExperimentalVMOptions` and `-XX:+UseCGroupMemoryLimitForHeap` JVM options will be
automatically provided when launching any component in order to make the JVM adhere to the memory
limits imposed by Docker. For more information, see [this article][jvm-docker-article].

Additional JVM options can be set through the `SP_JAVA_OPTS` environment variable.

See also the [setup guide](Snowplow-Docker-Setup).

[base-image]: https://github.com/snowplow/snowplow-docker/tree/master/base
[alpine-image]: https://github.com/docker-library/openjdk/blob/master/8-jre/alpine/Dockerfile

[iglu-server]: https://github.com/snowplow/iglu/wiki/Iglu-server

[dumb-init]: https://github.com/Yelp/dumb-init
[su-exec]: https://github.com/ncopa/su-exec

[jvm-docker-article]: https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits

