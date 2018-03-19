[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 1: setup a Collector](Setting-up-a-Collector) » Setting up the Scala Stream Collector

## Overview of the Scala Stream Collector

The Scala Stream Collector allows near-real-time processing (Enrichment, Storage, Analytics) of a
Snowplow raw event stream. Snowplow raw events can be sunk to either [Amazon Kinesis][kinesis],
[Google PubSub][pubsub], [Apache Kafka][kafka], [NSQ][nsq] or to `stdout` for a custom stream
collection process.

For more information on the architecture of the Scala Stream Collector, please see [[Scala Stream Collector]].

## Contents

Setting up the Scala Stream Collector is a 3 step process:

1. [[Install the Scala Stream Collector]]
2. [[Configure the Scala Stream Collector]]
3. [[Run the Scala Stream Collector]]

**Note**: If you're running on AWS, we recommend running all Snowplow AWS operations through an IAM
user with the bare minimum permissions required to run Snowplow. Please see our
[IAM user setup page](IAM-setup) for more information on doing this.

[kinesis]: http://aws.amazon.com/kinesis/
[kafka]: http://kafka.apache.org/
[nsq]: http://nsq.io
[pubsub]: https://cloud.google.com/pubsub/
