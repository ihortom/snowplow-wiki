<a name="top" />

[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Configurable enrichments](Configurable-enrichments) » IP lookups enrichment

You can find documentation for version 1-0-0 of the enrichment in
[the dedicated wiki page](IP-lookups-enrichment-1-0-0).

### Compatibility

JSON Schema   [iglu:com.snowplowanalytics.snowplow/ip_lookups/jsonschema/2-0-0][schema]
Compatibility R103
Data provider [MaxMind][maxmind]

### Overview

This enrichment uses MaxMind databases to look up useful data based on a user's IP address.

There are five possible fields you can add to the "parameters" section of the enrichment
configuration JSON: "geo", "isp", "domain", and "connectionType". Each of these corresponds to
looking up information one of five MaxMind databases, and so needs to have two inner fields:

* The `database` field contains the name of the database file.
* The `uri` field contains the URI of the bucket in which the database file is found. Can have either http: (for publically available MaxMind files) or s3: (for commercial MaxMind files) as the scheme. Must *not* end with a trailing slash.

The below table describes the five types of lookup. This version of the enrichment only works with
the new binary formats (.MMDB) which you should have access to with any subscription to
[MaxMind][maxmind].

| **Field name**   | **MaxMind Database name**     | **Lookup description**                             | **Accepted database filenames**                   | **Fields populated** |
|-----------------:|:------------------------------|:---------------------------------------------------|:--------------------------------------------------|:---------------------|
| `"geo"`          | [GeoIP2 City][geoip2-city] or [GeoLite2 City][geolite2-city] | Information related to geographic location         | `"GeoLite2-City.mmdb"` or `"GeoIP2-City.mmdb"`           | `geo_country`, `geo_region`, `geo_city`, `geo_zipcode`, `geo_latitude`, `geo_longitude`, and `geo_region_name` |
| `"isp"`          | [GeoIP2 ISP][geoip2-isp]                   | Internet Service Provider                          | `"GeoIP2-ISP.mmdb"`                                  | `ip_isp`          |
| `"domain"`       | [GeoIP2 Domain][geoip2-domain]                | Second level domain name associated with IP address | `"GeoIP2-Domain.mmdb"`                               | `ip_domain`       |
| `"connectionType"`     | [GeoIP2 Connection Type][geoip2-connection-type]              | Estimated connection type                         | `"GeoIP2-Connection-Type.mmdb"` | `ip_netspeed`     |

**Field name** is the name of the field in the ip_lookups enrichment configuration JSON which you should include if you wish to use that type of lookup. That field should have two subfields: "uri" and "database".

**MaxMind Database name** is the name of the database which that lookup uses.

**Lookup description** describes the lookup.

**Accepted database filenames** are the strings which are allowed in the "database" subfield. If the file name you provide is not one of these, the enrichment JSON will fail validation.

**Fields populated** are the names of the database fields which the lookup fills.

For each of these services you wish to use, add a corresponding field to the enrichment JSON. The fields names you should use are "geo", "isp", "organization", "domain", and "netspeed".

### Examples

Here is a maximalist example configuration JSON, which performs all five types of lookup using the MaxMind commercial files:

```json
{
	"schema": "iglu:com.snowplowanalytics.snowplow/ip_lookups/jsonschema/2-0-0",

	"data": {

		"name": "ip_lookups",
		"vendor": "com.snowplowanalytics.snowplow",
		"enabled": true,
		"parameters": {
			"geo": {
				"database": "GeoIP2-City.mmdb",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			},
			"isp": {
				"database": "GeoIP2-ISP.mmdb",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			},
			"domain": {
				"database": "GeoIP2-Domain.mmdb",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			},
			"connectionType": {
				"database": "GeoIP2-Connection-Type.mmdb",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			}
		}
	}
}
```

Here is a simpler example configuration with the free files:

```json
{
	"schema": "iglu:com.snowplowanalytics.snowplow/ip_lookups/jsonschema/2-0-0",

	"data": {

		"name": "ip_lookups",
		"vendor": "com.snowplowanalytics.snowplow",
		"enabled": true,
		"parameters": {
			"geo": {
				"database": "GeoLite2-City.mmdb",
				"uri": "http://snowplow-hosted-assets.s3.amazonaws.com/third-party/maxmind"
			}
		}
	}
}
```

This example uses the free GeoLite2-City database hosted by Snowplow.

### Data sources

The only input value for this enrichment comes from `ip` parameter, which maps to `user_ipaddress` field in `atomic.events` table.

### Algorithm

This enrichment uses 3rd party, [MaxMind][maxmind], service to look up data associated with the IP address. MaxMind offer industry-leading IP intelligence data updated monthly.

### Data generated

Below is the summary of the fields in `atomic.events` table driven by the result of this enrichment (no dedicated table).

Field | Purpose
:---|:---
`geo_country` | Country of IP origin
`geo_region` | Region of IP origin
`geo_city` | City of IP origin
`geo_zipcode` | Zip (postal) code
`geo_latitude` | An approximate latitude (coordinates)
`geo_longitude` | An approximate longitude (coordinates)
`geo_region_name` | Region
`ip_isp` | ISP name
`ip_organization` | Organization name for larger networks
`ip_domain` | Second level domain name
`ip_netspeed` | Indication of connection type (dial-up, cellular, cable/DSL)


[schema]: http://iglucentral.com/schemas/com.snowplowanalytics.snowplow/ip_lookups/jsonschema/2-0-0
[maxmind]: https://www.maxmind.com/en/home
[geoip2-city]: https://www.maxmind.com/en/geoip2-city?rld=snowplow
[geolite2-city]: https://dev.maxmind.com/geoip/geoip2/geolite2/?rld=snowplow
[geoip2-isp]: https://www.maxmind.com/en/geoip2-isp-database?rld=snowplow
[geoip2-domain]: https://www.maxmind.com/en/geoip2-domain-name-database?rld=snowplow
[geoip2-connection-type]: https://www.maxmind.com/en/geoip2-connection-type-database?rld=snowplow
