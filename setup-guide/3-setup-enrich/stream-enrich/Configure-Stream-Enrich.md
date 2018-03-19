<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Step 3.2: setting up Stream Enrich](setting-up-stream-enrich) » [[Install Stream Enrich]] » Configure Stream Enrich

**This documentation is for version 0.11.x to 0.15.x of Stream Enrich. Documentation for other versions is available:**

* **[Version 0.6.0-0.10.0][v0.10]**

## Basic configuration

### Template

Download a template configuration file from GitHub: [config.hocon.sample][app-conf].

Now open the `config.hocon.sample` file in your editor of choice.

### Monitoring

You can also now include Snowplow Monitoring in the application. This is setup through an optional
section at the bottom of the config. You will need to ammend:

+ `monitoring.snowplow.collectorUri` insert your snowplow collector URI here.
+ `monitoring.snowplow.appId` the app-id used in decorating the events sent.

If you do not wish to include Snowplow Monitoring please remove the entire `monitoring` section from
the config.

## Resolver configuration

You will also need a JSON configuration for the Iglu resolver used to look up JSON schemas. A sample configuration is available [here][resolver.json.sample].

### Configuring enrichments

You may wish to use Snowplow's configurable enrichments. To do this, create a directory of
enrichment JSONs. For each configurable enrichment you wish to use, the enrichments directory should
contain a .json file with a configuration JSON for that enrichment. When you come to run Stream
Enrich you can then pass in the filepath to this directory using the --enrichments option.

Sensible default configuration enrichments are available on GitHub:
[3-enrich/config/enrichments][enrichment-json-examples].

See the documentation on [configuring enrichments][configuring-enrichments] for details on the
available enrichments.

### Storage in DynamoDB

Rather than keeping the resolver JSON in a local file, you can store it in a [DynamoDB][ddb] table.

### GeoLiteCity databases

For the `ip_lookups` enrichment you can manually download the latest version of the [MaxMind GeoLiteCity database][geolite] jarfile directly from our Hosted Assets bucket on Amazon S3 - please see our [[Hosted assets]] page for details.

If you do not have the databases downloaded prior to running but still have the `ip_lookups`
enrichment activated then these databases will be downloaded automatically and saved in the working
directory. Not that although Spark Enrich can download databases whose URIs begin with "s3://" from
S3, Stream Enrich can't.

Next: [[Run Stream Enrich]]

[v0.10]: https://github.com/snowplow/snowplow/wiki/Configure-Stream-Enrich-0-10

[geolite]: http://dev.maxmind.com/geoip/legacy/geolite/?rld=snowplow
[app-conf]: https://raw.githubusercontent.com/snowplow/snowplow/master/3-enrich/stream-enrich/examples/config.hocon.sample
[enrichment-json-examples]: https://github.com/snowplow/snowplow/tree/master/3-enrich/config/enrichments
[configuring-enrichments]: https://github.com/snowplow/snowplow/wiki/Configurable-enrichments
[resolver.json.sample]: https://raw.githubusercontent.com/snowplow/snowplow/master/3-enrich/config/iglu_resolver.json
[ddb]: http://aws.amazon.com/dynamodb/
