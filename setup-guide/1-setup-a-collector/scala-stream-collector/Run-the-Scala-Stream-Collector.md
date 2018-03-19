[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 1: setup a Collector](Setting-up-a-Collector) » [[Setting up the Scala Stream Collector]] » [[Install the Scala Stream Collector]] » [[Configure the Scala Stream Collector]] » Run the Scala Stream Collector

## Running

The Scala Stream Collector is a jarfile. Simply provide the configuration file as a parameter:

    $ java -jar snowplow-stream-collector-[targeted platform]-[version].jar --config my.conf

Where `targeted platform` can be one of:

- kinesis
- google-pubsub
- kafka
- nsq
- stdout

This will start the collector as a HTTP service and write serialized Thrift records to either
`stdout`, Kinesis, Google PubSub, Kafka or NSQ, depending on the JAR.

## Healthcheck

Pinging the collector on the /health path should return a 200 OK response:

    $ curl http://localhost:8080/health

## All done?

You have setup the Scala Stream collector! You are now ready to [setup a tracker](Setting-up-a-Tracker).

Return to the [setup guide](Setting-up-Snowplow).
