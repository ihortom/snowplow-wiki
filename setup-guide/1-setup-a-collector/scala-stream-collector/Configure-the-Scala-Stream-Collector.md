[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 1: setup a Collector](Setting-up-a-Collector) » [[Setting up the Scala Stream Collector]] » [[Install the Scala Stream Collector]] » Configure the Scala Stream Collector

**This documentation refers to version 0.12.0 and 0.13.0 of the Scala Stream Collector**

**[Version 0.10.0][v0.10]**
**[Version 0.11.0][v0.11]**

The Scala Stream Collector has a number of configuration options available.

## Basic configuration

### Template

Download a template configuration file from GitHub: [config.hocon.sample][app-conf].

Now open the `config.hocon.sample` file in your editor of choice.

### Stream configuration

You will need to put in the names of your `good` and `bad` streams:

+ `collector.streams.good`: The name of the good input stream of the tool which you choose as a sink. This is stream where the events which have successfully been collected will be stored in.
+ `collector.streams.bad`: The name of the bad input stream of the tool which you choose as a sink. This is stream where the events that are too big (w.r.t Kinesis 1MB limit) will be stored in.
+ `collector.streams.useIpAddressAsPartitionKey`: Whether to use the incoming event's ip as the partition key for the good stream/topic

### HTTP settings

Also verify the settings of the HTTP service:

+ `collector.interface`
+ `collector.port`

### Buffer settings

You will also need to set appropriate limits for:

+ `collector.streams.buffer.byteLimit`
+ `collector.streams.buffer.recordLimit`
+ `collector.streams.buffer.timeLimit`

## Additional configuration options

### 1. Sinks

The `collector.streams.sink.enabled` setting determines which of the supported sinks to write raw
events to:
+ `"kinesis"` for writing Thrift-serialized records and error rows to a Kinesis stream
+ `"stdout"` for writing Base64-encoded Thrift-serialized records and error rows to stdout and stderr respectively
+ `"kafka"` for writing Thrift-serialized records and error rows to Kafka stream
+ `"nsq"` for writing Thrift-serialized records and error rows to NSQ topic

If you switch to `"stdout"`, we recommend setting 'akka.loglevel = OFF' and 'akka.loggers = []' to prevent Akka debug information from polluting your event stream on stdout.

You should fill the rest of the `collector.streams.sink` section according to your selection as a
sink.

To use `stdout` as a sink comment everything in the `collector.streams.sink` but
`collector.streams.sink.enabled` which should be set to `stdout`.

### 2. Setting the P3P policy header

The P3P policy header is set in `collector.p3p`, and
if not set, the P3P policy header defaults to:

	policyRef="/w3c/p3p.xml", CP="NOI DSP COR NID PSA OUR IND COM NAV STA"

### 3. Setting the domain name

Set the cookie name using the `collector.cookie.name` setting. To maintain backward compatibility with earlier versions of the collector, use the string "sp" as the cookie name.

Setting the domain name in `collector.cookie.domain` can be useful if you want to make the cookie accessible to other applications on your domain. In our example above, for example, we've setup the collector on `collector.snplow.com`. If we do not set a domain name, the cookie will default to this domain. However, if we set it to `.snplow.com`, that cookie will be accessible to other applications running on `*.snplow.com`.

Please, refer to [RFC 6265](https://tools.ietf.org/html/rfc6265#section-5.1.3) for the domain matching rules.

### 4. Setting the cookie duration

The cookie expiration duration is set in `collector.cookie.expiration`. If no value is provided, cookies set the default to expire after one year (i.e. 365 days). If you don't want to set a third party cookie at all it could be disabled by setting `collector.cookie.enabled` to `false`. Alternatively, it could be achieved if `collector.cookie.expiration` is set to 0 (from version 0.4.0).

» Next: [[Run the Scala Stream collector]]

[v0.10]: https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector-v0.10
[v0.11]: https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector-v0.11
[app-conf]: https://raw.githubusercontent.com/snowplow/snowplow/master/2-collectors/scala-stream-collector/examples/config.hocon.sample
