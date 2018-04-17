<a name="top" />

[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 2b: setup a Webhook**](Setting-up-a-webhook) > [[Iglu webhook setup]]

## Contents

- 1 [Overview](#overview)
  - 1.1 [Compatibility](#compat)
- 2 [Setup](#setup)
  - 2.1 [Your webhook](#setup-webhook)
  - 2.2 [Snowplow Redshift](#setup-redshift)

<a name="overview" />

## 1. Overview

This webhook adapter lets you track events sent via a `GET` or `POST` request containing an [Iglu][iglu]-compatible event payload.

You can use this adapter with vendors who allow you define your **own** event types for "postback". An example of a vendor who supports this is [AD-X Tracking][adxtracking-website].

For the technical implementation, see [[Iglu webhook adapter]].

<a name="compat" />

### 1.1 Compatibility

* [Snowplow 0.9.11][snowplow-0.9.11]+ (all collectors)
* Iglu self-describing JSON
* Iglu Webhook `POST` support - [Snowplow R83 Bald Eagle][snowplow-r83]

<a name="setup" />

## 2. Setup

Integrating Iglu-compatible webhooks into Snowplow is a two-stage process:

1. Configure your third-party system to send Iglu-compatible events to Snowplow
2. (Optional) Setup the appropriate JSON Schema, JSON Paths file and Redshift table definition for each Iglu-compatible event you are sending through

<a name="setup-webhook" />

## 2.1 Your webhook

The Iglu webhook adapter supports events send in as `GET` and `POST` requests.

### 2.1.1 Path

We use a special path to tell Snowplow that these events should be parsed as Iglu self-describing JSON events, thus:

```
http://<collector host>/com.snowplowanalytics.iglu/v1?schema=<iglu schema uri>&...
```

### 2.1.2 Required fields

You can send in whatever name-value pairs on the querystring that make sense for your event, but you **must** also include a `schema` parameter, which is set to a valid Iglu self-describing schema URI, such as:

```
iglu:com.acme.postbacks/install_error/jsonschema/1-0-0
```

### 2.1.3 Optional fields

If you want to specify which app these events belong to, add an `aid` parameter as taken from the [Snowplow Tracker Protocol][tracker-protocol]:

```
...&aid=<company code>&...
```

You can also manually override the event's `platform` parameter like so:

```
...&p=<platform code>&...
```

Supported platform codes can again be found in the [Snowplow Tracker Protocol][tracker-protocol]; if not set, then the value for `platform` will default to `srv` for a server-side application.

### 2.1.4 Example `GET` request

Here is an example of an Iglu-compatible event sent as a `GET` request, broken out onto multiple lines to make it easier to read:

```
http://snplow.acme.com/com.snowplowanalytics.iglu/v1
  ?schema=iglu%3Acom.acme%2Fcampaign%2Fjsonschema%2F1-0-0
  &aid=mobile-attribution&p=mob
  &user=6353af9b-e288-4cf3-9f1c-b377a9c84dac&name=download&publisher_name=Organic&source=&tracking_id=&ad_unit=
```

This will be converted by the Iglu webhook adapter into a self-describing JSON looking like this:

```json
{
  "schema":"iglu:com.acme/campaign/jsonschema/1-0-0",
  "data": {
    "name":"download",
    "source":null,
    "ad_unit":null,
    "tracking_id":null,
    "publisher_name":"Organic",
    "user":"6353af9b-e288-4cf3-9f1c-b377a9c84dac"
  }
}
```

The Snowplow enriched event containing this JSON will include `app_id` set to "mobile-attribution" and `platform` set to "mob".

### 2.1.5 Example `POST` request

POST requests can be compiled in two different ways for the Iglu webhook:

* As a full SelfDescribing JSON in the body
* With a `?schema=<iglu schema uri>` in the querystring and a data JSON in the body

__NOTE__: For the event to be accepted the `Content-Type` must be either:

* `application/json`
* `application/json; charset=utf-8`

To send the first type:

```
http://snplow.acme.com/com.snowplowanalytics.iglu/v1 -d '{
  "schema":"iglu:com.acme/campaign/jsonschema/1-0-0",
  "data": {
    "name":"download",
    "source":null,
    "ad_unit":null,
    "tracking_id":null,
    "publisher_name":"Organic",
    "user":"6353af9b-e288-4cf3-9f1c-b377a9c84dac"
  }
}'
```

To send the second type:

```
http://snplow.acme.com/com.snowplowanalytics.iglu/v1?schema=iglu%3Acom.acme%2Fcampaign%2Fjsonschema%2F1-0-0 -d '{
  "name":"download",
  "source":null,
  "ad_unit":null,
  "tracking_id":null,
  "publisher_name":"Organic",
  "user":"6353af9b-e288-4cf3-9f1c-b377a9c84dac"
}'
```

As with the `GET` request above you can also attach extra information into the querystring to help describe your event.  The following parameters can be added:

* `aid=` : The application ID
* `p=` : The platform
* `nuid=` : The network user ID
* `eid=` : A custom event ID
* `ttm=` : The true timestamp
* `url=` : The page URL
* `cv=` : The context vendor (deprecated)

<a name="setup-redshift" />

## 2.2 Redshift

If you are running the Snowplow batch flow with Amazon Redshift, then you will need to define for each Iglu-compatible event:

* A JSON Schema
* A JSON Paths file
* A Redshift table definition

Creating and publishing these artifacts is out of scope for this setup page - for details please see the wiki page on [[Shredding]].

Please note that a limitation of this adapter is that all event properties will end up being "stringly typed" - the adapter has no way of knowing which parameters should be converted into numbers, booleans, date-times or similar. When defining your JSON Schema and Redshift table definition, please remember that specifying non-string types for any of the fields will cause shredding to fail.

And that's it - you should be ready now to start processing Iglu-compatible events through into Redshift.

[iglu]: https://github.com/snowplow/iglu

[adxtracking-website]: http://adxtracking.com/
[snowplow-0.9.11]: https://github.com/snowplow/snowplow/releases/tag/0.9.11
[snowplow-r83]: https://github.com/snowplow/snowplow/releases/tag/r83-bald-eagle

[tracker-protocol]: https://github.com/snowplow/snowplow/wiki/snowplow-tracker-protocol#1-common-parameters-platform-and-event-independent
