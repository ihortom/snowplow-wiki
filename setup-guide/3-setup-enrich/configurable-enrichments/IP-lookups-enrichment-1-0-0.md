<a name="top" />

[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Configurable enrichments](Configurable-enrichments) » IP lookups enrichment

### Compatibility

JSON Schema   [iglu:com.snowplowanalytics.snowplow/ip_lookups/jsonschema/1-0-0][schema]
Compatibility 0.9.6+
Data provider [MaxMind][maxmind]

### Overview

This enrichment uses MaxMind databases to look up useful data based on a user's IP address.

There are five possible fields you can add to the "parameters" section of the enrichment configuration JSON: "geo", "isp", "organization", "domain", and "netspeed". Each of these corresponds to looking up information one of five MaxMind databases, and so needs to have two inner fields:

* The `database` field contains the name of the database file.
* The `uri` field contains the URI of the bucket in which the database file is found. Can have either http: (for publically available MaxMind files) or s3: (for commercial MaxMind files) as the scheme. Must *not* end with a trailing slash.

The below table describes the five types of lookup. Note Snowplow only works with the legacy binary formats (.DAT) which you should have access to with any subscription to [MaxMind][maxmind].

| **Field name**   | **MaxMind Database name**     | **Lookup description**                             | **Accepted database filenames**                   | **Fields populated** |
|-----------------:|:------------------------------|:---------------------------------------------------|:--------------------------------------------------|:---------------------|
| `"geo"`          | [GeoIPCity][geoipcity] or [GeoLiteCity][geolitecity] | Information related to geographic location         | `"GeoLiteCity.dat"` or `"GeoIPCity.dat"`           | `geo_country`, `geo_region`, `geo_city`, `geo_zipcode`, `geo_latitude`, `geo_longitude`, and `geo_region_name` |
| `"isp"`          | [GeoIP ISP][geoipisp]                   | Internet Service Provider                          | `"GeoIPISP.dat"`                                  | `ip_isp`          |
| `"organization"` | [GeoIP Organization][geoiporg]          | Organization name for larger networks              | `"GeoIPOrg.dat"`                                  | `ip_organization` |
| `"domain"`       | [GeoIP Domain][geoipdomain]                | Second level domain name associated with IP address | `"GeoIPDomain.dat"`                               | `ip_domain`       |
| `"netspeed"`     | [GeoIP Netspeed][geoipnetspeed]              | Estimated connection speed                         | `"GeoIPNetSpeed.dat"` or `"GeoIPNetSpeedCell.dat"` | `ip_netspeed`     |

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
	"schema": "iglu:com.snowplowanalytics.snowplow/ip_lookups/jsonschema/1-0-0",

	"data": {

		"name": "ip_lookups",
		"vendor": "com.snowplowanalytics.snowplow",
		"enabled": true,
		"parameters": {
			"geo": {
				"database": "GeoIPCity.dat",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			},
			"isp": {
				"database": "GeoIPISP.dat",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			},
			"organization": {
				"database": "GeoIPOrg.dat",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			},
			"domain": {
				"database": "GeoIPDomain.dat",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			},
			"netspeed": {
				"database": "GeoIPNetSpeedCell.dat",
				"uri": "s3://my-private-bucket/third-party/maxmind"
			}
		}
	}
}
```

Here is a simpler example configuration (which exactly duplicates the behaviour of Snowplow 0.9.5):

```json
{
	"schema": "iglu:com.snowplowanalytics.snowplow/ip_lookups/jsonschema/1-0-0",

	"data": {

		"name": "ip_lookups",
		"vendor": "com.snowplowanalytics.snowplow",
		"enabled": true,
		"parameters": {
			"geo": {
				"database": "GeoLiteCity.dat",
				"uri": "http://snowplow-hosted-assets.s3.amazonaws.com/third-party/maxmind"
			}
		}
	}
}
```

This example uses the free GeoLiteCity database hosted by Snowplow.

### Data sources

The only input value for this enrichment comes from `ip` parameter, which maps to `user_ipaddress` field in `atomic.events` table.

### Algorithm

This enrichment uses 3rd party, [MaxMind][maxmind], service to look up data associated with the IP address. MaxMind offer industry-leading IP intelligence data updated weekly.

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


[schema]: http://iglucentral.com/schemas/com.snowplowanalytics.snowplow/ip_lookups/jsonschema/1-0-0
[maxmind]: https://www.maxmind.com/en/home
[geoipcity]: https://www.maxmind.com/en/geoip2-city?rld=snowplow
[geolitecity]: http://dev.maxmind.com/geoip/legacy/geolite/?rld=snowplow
[geoipisp]: https://www.maxmind.com/en/isp?rld=snowplow
[geoiporg]: https://www.maxmind.com/en/organization?rld=snowplow
[geoipdomain]: https://www.maxmind.com/en/domain?rld=snowplow
[geoipnetspeed]: https://www.maxmind.com/en/netspeed?rld=snowplow
