<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Step 3.2: setting up Stream Enrich](setting-up-stream-enrich) » [[Install Stream Enrich]] » [[Configure Stream Enrich]] » Run Stream Enrich

**This documentation is for version 0.11.x, 0.12.x and 0.13.x of Stream Enrich. Documentation for other versions is available:**

- [Stream Enrich v0.5.0 - v0.10.0][v010]

## Running

Stream Enrich is a jarfile. Simply provide the configuration file as a parameter:

    $ java -jar snowplow-stream-enrich-0.13.0.jar --config my.conf --resolver file:resolver.json

This will start the Stream Enrich app to read raw events from Kinesis, Kafka, or NSQ and write
enriched events back to Kinesis, Kafka or NSQ.

If you are using configurable enrichments, provide the path to your enrichments directory as a
parameter:

    $ java -jar snowplow-stream-enrich-0.13.0.jar --config my.conf --resolver file:resolver.js --enrichments file:path/to/enrichments

If you are storing the resolver and/or enrichments in DynamoDB, use the "dynamodb:" prefix in place
of the "file:" prefix:

    $ java -jar snowplow-stream-enrich-0.13.0.jar --config my.conf --resolver dynamodb:eu-west-1/ConfigurationTable/resolver --enrichments dynamodb:eu-west-1/ConfigurationTable/enrichment_

The above command that the enrichments and resolver are stored in a table named ConfigurationTable
in eu-west-1, that the hash key for that table is "id", that the resolver JSON is stored in an item
whose hash key has value "resolver", and the enrichments are stored in items whose hash keys have
values beginning with "enrichment_".

### Configuring the log level

Stream Enrich uses [slf4j logging][logging]:

    $ java -Dorg.slf4j.simpleLogger.defaultLogLevel=debug \
        -jar snowplow-stream-enrich-0.13.x.jar --config my.conf --resolver file:resolver.json

This will also affect messages logged by the [Kinesis Client Library][kcl](which Stream Enrich uses
to read from Kinesis.)

## All done?

You have setup Stream Enrich! You are now ready to [setup alternative data stores](Setting-up-alternative-data-stores).

Return to the [setup guide](Setting-up-Snowplow).

[v010]: https://github.com/snowplow/snowplow/wiki/Run-Stream-Enrich-0-10.md

[logging]: http://www.slf4j.org/api/org/slf4j/impl/SimpleLogger.html
[kcl]: https://github.com/awslabs/amazon-kinesis-client

[scala-out]: https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector#2-sinks
[scala-template]: https://github.com/snowplow/snowplow/wiki/Configure-the-Scala-Stream-Collector#template
[kinesis-in]: https://github.com/snowplow/snowplow/wiki/Configure-Scala-Kinesis-Enrich#source
[kinesis-template]: https://github.com/snowplow/snowplow/wiki/Configure-Scala-Kinesis-Enrich#template
