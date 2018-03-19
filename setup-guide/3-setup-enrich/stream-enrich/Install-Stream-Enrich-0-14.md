<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Step 3.2: setting up Stream Enrich](setting-up-stream-enrich) » Install Stream Enrich

**This documentation is for version 0.11.x to 0.14.x of Stream Enrich. Documentation for other versions is available:**

**[Version 0.6.0-0.10.0][v0.10]**

## 1. Dependencies

You will need version 8 (aka 1.8) of the Java Runtime Environment installed.

## 2. Getting the jar

You can choose to either:

1. Download the Stream Enrich jarfile, _or:_
2. Compile it from source

### 2.1 Download the jarfile

To get a local copy, you can download the executable jarfile directly from our Hosted Assets bucket
on Amazon S3 - please see our [[Hosted assets]] page for details.

### 2.2 Compile from source

Alternatively, you can build it from the source files. To do so, you will need [scala][scala] and
[sbt][sbt] installed.

To do so, clone the Snowplow repo:

	$ git clone https://github.com/snowplow/snowplow.git

Navigate into the Stream Enrich folder:

	$ cd 3-enrich/stream-enrich

Use `sbt` to resolve dependencies, compile the source, and build an [assembled][assembly] fat JAR
file with all dependencies.

	$ sbt assembly

The `jar` file will be saved as `snowplow-stream-enrich-0.13.0.jar` in the `target/scala-2.11`
subdirectory - it is now ready to be deployed.

Next: [[Configure Stream Enrich]]

[scala]: http://scala-lang.org/
[sbt]: http://www.scala-sbt.org/
[assembly]: https://github.com/sbt/sbt-assembly
[v0.10]: https://github.com/snowplow/snowplow/wiki/Install-Stream-Enrich-0-10
