<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Step 3: Setting up Enrich](Setting-up-enrich) > Configurable enrichments

Snowplow offers the option to configure certain enrichments. This is done using configuration JSONs. When running EmrEtlRunner or Kinesis Enrich, the `--enrichments` argument should be populated with the filepath of a directory containing your configuration JSONs. The files in the directory should be named "{{name-of-enrichment}}.json", where "name-of-enrichment" is the value of the "name" field in the JSON.

Each enrichment has a JSON schema against which it is validated. (For more information on how Snowplow uses JSON schemas, see [this blog post][snowplow-schemas]) The enrichment JSONs follow a common pattern:

```json
{
	"schema": "iglu:((self-describing JSON schema for the enrichment))",

	"data": {

		"name": "enrichment name",
		"vendor": "enrichment vendor",
		"enabled": true / false,
		"parameters": {
			((enrichment-specific settings))
		}
	}
}
```

For each enrichment, the JSON used to configure the enrichment has:
* A `name` field containing the name of the enrichment
* A `vendor` field containing the name of the company which created the enrichment
* An `enabled` field which can be set to `false` to ignore the enrichment
* A `parameters` field, containing the information needed to configure the enrichment

The `enabled` and `parameters` fields are the ones which you may wish to customize.

These are the available configurable enrichments:

[[IP anonymization enrichment]]  
[[IP lookups enrichment]]  
[[Campaign attribution enrichment]]  
[[Currency conversion enrichment]]  
[[JavaScript script enrichment]]  
[[referer-parser enrichment]]  
[[ua-parser enrichment]]  
[[user-agent-utils enrichment]]  
[[Event fingerprint enrichment]]  
[[Cookie extractor enrichment]]  
[[Weather enrichment]]  
[[API Request enrichment]]  
[[HTTP Header extractor enrichment]]  
[[SQL Query enrichment]]  
[[PII Pseudonymization Enrichment]]  

Snowplow ships with a set of sensible default configurations for the configurable enrichments. You can browser them on GitHub: [3-enrich/config/enrichments][enrichment-json-examples].

[enrichment-json-examples]: https://github.com/snowplow/snowplow/tree/master/3-enrich/config/enrichments/
[snowplow-schemas]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
