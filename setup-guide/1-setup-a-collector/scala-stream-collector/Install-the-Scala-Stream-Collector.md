[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 1: setup a Collector](Setting-up-a-Collector) » [[Setting up the Scala Stream Collector]] » Install the Scala Stream Collector

## 1. Dependencies

You will need version 8 (aka 1.8) of the Java Runtime Environment installed.

## 2. Installing the jarfile

You can choose to either:

1. Download the Scala Stream collector jarfile, _or:_
2. Compile it from source

## 2.1 Download the jarfile

To get a local copy, you can download the jarfile directly from our hosted assets bucket on Amazon
S3 - please see Section 6 in our [[Hosted assets]] page for details.

## 2.2 Compile from source

Alternatively, you can build it from the source files. To do so, you will need [scala][scala] and
[sbt][sbt] installed.

To do so, clone the Snowplow repo:

	$ git clone https://github.com/snowplow/snowplow.git

Navigate into the Scala Stream collector folder:

	$ cd 2-collectors/scala-stream-collector

Use `sbt` to resolve dependencies, compile the source, and build an [assembled][assembly] fat JAR
file with all dependencies.

	$ sbt "project *targeted platform*" assembly

where `targeted platform` can be:

- kinesis
- pubsub
- kafka
- nsq
- stdout

The `jar` file will be saved as `snowplow-scala-collector-[targeted platform]-[version].jar` in the
`[targeted platform]/target/scala-2.11` subdirectory - it is now ready to be deployed.

Next: [[Configure the Scala Stream Collector]]

[s3-download]: https://github.com/snowplow/snowplow/wiki/Hosted-assets
[scala]: http://scala-lang.org/
[sbt]: http://www.scala-sbt.org/
[assembly]: https://github.com/sbt/sbt-assembly
