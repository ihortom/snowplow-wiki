<a name="top" />

[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Configurable enrichments](Configurable-enrichments) » PII Pseudonymization Enrichment

### Compatibility

JSON Schema   [iglu:com.snowplowanalytics.snowplow.enrichments/pii_enrichment_config/jsonschema/1-0-0][schema]<br/>
Compatibility R100 <br/>
Data provider None (Internal transformation)<br/>

### Overview

The PII Enrichment provides capabilities for Snowplow operators to better protect the privacy rights of data subjects. The obligations of handlers of Personally Identifiable Information (PII) data under GDPR have been outlined on the [EU GDPR website][gdpr].

The r100 release of the PII Enrichment provides a way to pseudonymize fields within Snowplow enriched events. You can configure the enrichment to pseudonymize any of the following datapoints:

1. Any of the “first-class” fields which are part of the Canonical event model, are scalar fields containing a single string and have been identified as being potentially sensitive
2. Any of the properties within the JSON instance of a Snowplow self-describing event or context (wherever that context originated). You simply specify the Iglu schema to target and a JSON Path to identify the property or properties within to pseudonomize

In addition, you must specify the “strategy” that will be used in the pseudonymization. Currently the available strategies involve hashing the PII, using one of the following algorithms:

* `MD2`, the 128-bit algorithm [MD2][md2] (not-recommended due to performance reasons see [RFC6149][rfc6149])
* `MD5`, the 128-bit algorithm [MD5][md5]
* `SHA-1`, the 160-bit algorithm [SHA-1][sha-1]
* `SHA-256`, 256-bit variant of the [SHA-2][sha-2] algorithm
* `SHA-384`, 384-bit variant of the [SHA-2][sha-2] algorithm
* `SHA-512`, 512-bit variant of the [SHA-2][sha-2] algorithm

### Prerequisite checks

In configuring the PII enrichment you will need to check that all fields comply with the following two checks:

#### Format
Always check the underlying JSON Schema to avoid accidentally invalidating an entire event using the PII Enrichment. Specifically, you should check the field definitions of the fields for any constraints that hold under plaintext but not when the field is hashed, such as field length and format.

The scenario to avoid is as follows:

* You have a `customerEmail` property in a JSON Schema which must validate with <br/>`format: email`
* You apply the PII Enrichment to hash that field
* The enriched event *is* successfully emitted from Stream Enrich...
* **However**, a downstream process (e.g. RDB Shredder) which validates the now-pseudonymized event will **reject** the event, as the hashed value is no longer in an email format

#### Length

The same issue can happen with properties with enforced string lengths - note that all of the currently supported pseudonymization functions will generate hashes of up to **128 characters** (in the case of SHA-512); be careful if the JSON Schema enforces a shorter length, as again the event will fail downstream validation.

### Example

Here is an example configuration:

```json
{
  "schema": "iglu:com.snowplowanalytics.snowplow.enrichments/pii_enrichment_config/jsonschema/1-0-0",
  "data": {
    "vendor": "com.snowplowanalytics.snowplow.enrichments",
    "name": "pii_enrichment_config",
    "enabled": true,
    "parameters": {
      "pii": [
        {
          "pojo": {
            "field": "user_id"
          }
        },
        {
          "pojo": {
            "field": "user_fingerprint"
          }
        },
        {
          "json": {
            "field": "unstruct_event",
            "schemaCriterion": "iglu:com.mailchimp/subscribe/jsonschema/1-0-*",
            "jsonPath": "$.data.['email', 'ip_opt']"
          }
        }
      ],
      "strategy": {
        "pseudonymize": {
          "hashFunction": "SHA-256"
        }
      }
    }
  }
}
```

The configuration above is for a Snowplow pipeline that is receiving events from the Snowplow JavaScript Tracker, plus a Mailchimp webhook integration:

* The Snowplow JavaScript Tracker has been configured to emit events which includes the `user_id` and `user_fingerprint` fields
* The Mailchimp webhook (available since [release 0.9.11][release-0.9.11]) is emitting `subscribe` events (among other events, ignored for the purpose of this example)

With the above PII Enrichment configuration, then, you are specifying that:

* You wish for the `user_id` and `user_fingerprint` from the Snowplow Canonical event model fields to be hashed (the full list of supported fields for pseudonymization is viewable [in the enrichment configuration schema][pii-config-schema])
* You wish for the `data.email` and `data.ip_opt` fields from the Mailchimp `subscribe` event to be hashed, but only if the schema version begins with `1-0-`
* You wish to use the `SHA-256` variant of the algorithm for the pseudonymization

### Data sources

There is no external input to this enrichment.

### Data generated

There are no new fields generated. The enrichment replaces the existing fields with new, pseudonymous values according to the selected algorithm.

[schema]: http://iglucentral.com/schemas/com.snowplowanalytics.snowplow.enrichments/pii_enrichment_config/jsonschema/1-0-0
[gdpr]: https://www.eugdpr.org/
[release-0.9.11]: https://snowplowanalytics.com/blog/2014/11/10/snowplow-0.9.11-released-with-webhook-support/#mailchimp
[pii-config-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow.enrichments/pii_enrichment_config/jsonschema/1-0-0

[md2]: https://en.wikipedia.org/wiki/MD2_(cryptography)#MD2_hashes
[md5]: https://en.wikipedia.org/wiki/MD5#MD5_hashes
[sha-1]: https://en.wikipedia.org/wiki/SHA-1#Example_hashes
[sha-2]: https://en.wikipedia.org/wiki/SHA-2#Comparison_of_SHA_functions
[rfc6149]: https://tools.ietf.org/html/rfc6149
