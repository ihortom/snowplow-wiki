<a name="top" />

[**HOME**](Home) > [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow technical documentation) > [**Trackers**](trackers) > JavaScript Tracker Core


## Contents

- 1. [Overview](#overview)  
- 2. [Initialization](#init)  
  - 2.1 [Importing the module](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
- 3. [Adding extra data](#add-data)
- 4. [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Tracker method return values](#return-values)
  - 4.2 [`trackScreenView()`](#screen-view)
  - 4.3 [`trackPageView()`](#page-view)
  - 4.4 [`trackEcommerceTransaction()`](#ecommerce-transaction)
    - 4.4.1 [`TransactionItem`](#ecommerce-transactionitem)
  - 4.5 [`trackStructuredEvent()`](#struct-event)
  - 4.6 [`trackUnstructuredEvent()`](#unstruct-event)
- 5 [Sending event](#emitter)
  - 5.1 [Using a buffer](#buffer)
  - 5.2 [Choosing the HTTP method](#http-method)
  - 5.3 [Method of sending HTTP requests](#http-request)
- 6 [Payload](#payload)
- 7 [Logging](#logging)

<a name="overview" />
## 1. Overview

The [Snowplow JavaScript Tracker Core][repo] is an [npm][npm-page] module providing functionality common to both the server-side [Snowplow Node.js Tracker][server-tracker] and the client-side [Snowplow JavaScript Tracker][client-tracker]. It supports all the Snowplow event types. Custom contexts and timestamps can be added to all events.

It has two main types of method: setter methods and tracking methods. Setter methods like `setUserId`

<a name="init" />
## 2 Initialization

<a name="requiring" />
### 2.1 Requiring the module

Require the Snowplow Tracker Core module into your code like so:

```js
var snowplowTrackerCore = require('snowplow-tracker-core');
```

<a name="requiring" />
### 2.1 Creating a tracker core instance

The tracker core constructor takes two parameters: 

* A boolean value indicating whether unstructured events and custom contexts should be base 64 encoded (which defaults to `true`)
* An optional callback to be executed on every payload before it is returned

```js
var t = snowplowTrackerCore(false, console.log);
```

The above example would create a tracker core instance which doesn't use base 64 encoding and which logs all payloads it creates to the console.


<a name="setter-methods" />
## 3. Setter methods

The core instance maintains a dictionary called `payloadPairs` containing persistent name-value pairs which are added to every payload. You can interact with these through the following methods:

<a name="addPayloadPair" />
### 3.1 `addPayloadPair()`

Adds a single name-value pair to `payloadPairs`.

```js
t.addPayloadPair('myKey', 'myValue');
```

<a name="addPayloadDict" />
### 3.2 `addPayloadDict()`

Adds all the name-value pairs in a dictionary to `payloadPairs`.

```js
t.addPayloadDict({
	key1: 'value1',
	key2: 'value2'
});
```

<a name="resetPayloadPairs" />
### 3.3 `resetPayloadPairs()`

Creates a new dictionary of payload pairs, removing all pre-existing name-value pairs. If no argument is provided, creates an empty dictionary.

```js
t.resetPayloadPairs({
	key1: 'value1',
	key2: 'value2'
});
```

<a name="other-setter-methods" />
### 3.4 Other setter methods

For convenience, other setter methods are provided for certain fields. They all use `addPayloadPair`.

| **Method name**       | **Key added** | **Example**                             |
|----------------------:|:--------------|:----------------------------------------|
| `setTrackerVersion`   | `tv`          |`t.setTrackerVersion('js-3.0.0');`       |
| `setTrackerNamespace` | `tna`         |`t.setTrackerNamespace('cloudfront-1');` | 
| `setAppId`            | `aid`         |`t.setAppId('my-node-application');`     |
| `setPlatform`         | `p`           |`t.setPlatform('web');`                  |
| `setUserId`           | `uid`         |`t.setUserId('user-427')`                |
| `setScreenResolution` | `res`         |`t.setScreenResolution(800, 600)`        | 
| `setViewport`         | `vp`          |`t.setViewport(307,250)`                 |
| `setColorDepth`       | `cd`          |`t.setColorDepth(24)`                    |
| `setTimezone`         | `tz`          |`t.setTimezone('Europe/London')`         |
| `setIpAddress`        | `ip`          |`t.setIpAddress('37.347.12.457')`        |

<a name="example" />
### 3.5 Example

Some calls to setter methds, along with comments indicating the current state of the `payloadPairs` dictionary:

```js
var snowplowTrackerCore = require('snowplow-tracker-core');
var t = snowplowTrackerCore(); // {}
t.addPayloadPair('a', 'one');  // {a: 'one'}
t.addPayloadDict({
	b: 'two',
	c: 'three'
});                            // {a: 'one', b: 'two', c: 'three'}
t.resetPayloadPairs();         // {}
t.setPlatform('srv');          // {p: 'srv'}
t.setPlatform('pc');           // {p: 'pc'}
t.setViewport(600, 400);       // {p: 'pc', 'vp': 600x400}
t.resetPayloadPairs({
	d: 'four',
	e: 'five'
});                            // {d: 'four', e: 'five'}
```

<a name="example" />
### 4 Event tracking

<a name="common" />
### 4.1 Common features

Any method whose name starts with `"track..."` is a tracking method which creates a payload for a single Snowplow event. The tracking methods have certain features in common.

<a name="common" />
#### 4.1.1 Custom contexts

Each tracking method's penultimate argument is an optional array of custom context dictionaries.

<a name="common" />
#### 4.1.2 Timestamp argument

Each tracking method's final argument is an optional timestamp. It should be in milliseconds since the Unix epoch - the same format generated by `new Date().getTime()`. If you don't provide a timestamp for an event, the current time will be used.

<a name="common" />
#### 4.1.3 Return value

When called, a tracker method will assemble a payload dictionary based on the arguments passed to it and the `payloadPairs` object, and will add a [version 4 UUID][uuid-4]. If you provided a callback function when constructing the core instance, that function will be passed the payload. (For example, the callback could be used to convert the payload to a querystring and send it to a Snowplow collector.) The payload will then be returned.

<a name="common" />
### 4.2 `trackScreenView()`

| **Argument** | **Description**                     | **Required?** | **Type**                |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | Yes           | Non-empty string        |
| `id`         | Unique identifier for this screen   | No            | String                  |
| `context`    | Custom context                      | No            | Array                   |
| `tstamp`     | When the screen was viewed          | No            | Positive integer        |

<a name="common" />
### 4.2 `trackPageView()`

| **Argument** | **Description**                     | **Required?** | **Type**                |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `pageUrl`    | The URL of the page                 | Yes           | Non-empty string        |
| `pageTitle`  | The title of the page               | No            | String                  |
| `referrer`   | The address which linked to the page| No            | String                  |
| `context`    | Custom context                      | No            | Array                   |
| `tstamp`     | When the screen was viewed          | No            | Positive integer        |

<a name="common" />
### 4.3 `trackStructEvent()`

| **Argument** | **Description**                                                  | **Required?** | **Type**         |
|-------------:|:---------------------------------------------------------------  |:--------------|:-----------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | Non-empty string |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | Non-empty string |
| `label`      | A string to provide additional dimensions to the event data      | No            | String           |
| `property`   | A string describing the object or the action performed on it     | No            | String           |
| `value`      | A value to provide numerical data about the event                | No            | Number           |
| `context`     | Custom context for the event                                    | No            | Array            |
| `tstamp`     | When the structured event occurred                               | No            | Positive integer |

<a name="common" />
### 4.3 `trackUnstructEvent()`

| **Argument**   | **Description**                      | **Required?** | **Type**                |
|---------------:|:-------------------------------------|:--------------|:------------------------|
| `properties`   | The properties of the event          | Yes           | JSON                    |
| `context`      | Custom context for the event         | No            | Array                   |
| `tstamp`       | When the unstructured event occurred | No            | Positive integer        |

<a name="common" />
### 4.3 `trackEcommerceTransaction()`

| **Argument**  | **Description**                      | **Required?** | **Type**                 |
|--------------:|:-------------------------------------|:--------------|:-------------------------|
| `orderId`     | ID of the eCommerce transaction      | Yes           | Non-empty string         |
| `affiliation` | Transaction affiliation              | No            | String                   |
| `totalValue`  | Total transaction value              | Yes           | Number                   |
| `taxValue`    | Transaction tax value                | No            | Number                   |
| `shipping`    | Delivery cost charged                | No            | Number                   |
| `city`        | Delivery address city                | No            | String                   |
| `state`       | Delivery address state               | No            | String                   |
| `country`     | Delivery address country             | No            | String                   |
| `currency`    | Currency                             | No            | String                   |
| `context`     | Custom context                       | No            | Array                    |
| `tstamp`      | When the transaction event occurred  | No            | Positive integer         |

<a name="common" />
### 4.3 `trackEcommerceTransactionItem()`

| **Field**  | **Description**                     | **Required?** | **Type**                 |
|-----------:|:------------------------------------|:--------------|:-------------------------|
| `orderId`  | ID of the eCommerce transaction     | Yes           | Non-empty string         |
| `sku`      | Item SKU                            | Yes           | Non-empty string         |
| `name`     | Item name                           | No            | String                   |
| `category` | Item category                       | No            | String                   |
| `price`    | Item price                          | Yes           | Number                   |
| `quantity` | Item quantity                       | Yes           | Int                      |
| `currency` | Currency                            | No            | String                   |
| `context`  | Custom context for the event        | No            | Array                    |
| `tstamp`   | When the transaction event occurred | No            | Positive integer         |

<a name="common" />
### 4.3 `trackAdImpression()`

| **Name**       | **Description**  | **Required?**                           | **Type**
|---------------:|:--------------|:-------------------------------------------|:---------------------|
| `impressionId` | Identifier for the particular impression instance  | No             | string |
|    `costModel` | The cost model for the campaign: 'cpc', 'cpm', or 'cpa' | No             | string |
|         `cost` | Ad cost | No              |    number    |
|   `targetUrl`  | The destination URL  | No            |        string   |
|     `bannerId` | Adserver identifier for the ad banner (creative) being displayed | No             | string                |
|       `zoneId` | Adserver identifier for the zone where the ad banner is located | No             | string  |
| `advertiserID` | Adserver identifier for the advertiser which the campaign belongs to | No                 | string   |
|   `campaignId` | Adserver identifier for the ad campaign which the banner belongs to | No               |  string |

<a name="common" />
### 4.3 `trackAdClick()`

| **Name**       | **Description**                            | **Required?** | **Type**
|---------------:|:-------------------------------------------|:--------------|:-----------
|   `targetUrl`  | The destination URL  | Yes           |        string   |
|      `clickId` | Identifier for the particular click instance   | No            | string  |
|    `costModel` | The cost model for the campaign: 'cpc', 'cpm', or 'cpa' | No            |  string  |
|         `cost` | Ad cost   | No            |    number    |
|     `bannerId` | Adserver identifier for the ad banner (creative) being displayed  | No            |  string
|       `zoneId` | Adserver identifier for the zone where the ad banner is located  | No            | string
| `advertiserID` | Adserver identifier for the advertiser which the campaign belongs to   | No            |  string                      |
|   `campaignId` | Adserver identifier for the ad campaign which the banner belongs | No            |  to   |  string                       |

<a name="common" />
### 4.3 `trackAdConversion()`

| **Name**       | **Description**                            | **Required?** | **Type**             |
|---------------:|:-------------------------------------------|:--------------|:--------------------
| `conversionId` | Identifier for the particular conversion instance   | No            | string
|    `costModel` | The cost model for the campaign: 'cpc', 'cpm', or 'cpa'   | No            |  string
|         `cost` | Ad cost   | No            |    number    |
|     `category` | Conversion category                        | No            |    number                              |
|       `action` | The type of user interaction, e.g. 'purchase' | No            | string                           |
|     `property` | Describes the object of the conversion        | No            | string                           |
| `initialValue` | How much the conversion is initially worth   | No            | number                           |
| `advertiserID` | Adserver identifier for the advertiser which the campaign belongs to | No            |  string   |
|   `campaignId` | Adserver identifier for the ad campaign which the banner belongs to  | No            |  string  |

<a name="common" />
### 4.3 `trackLinkClick()`

| **Field**        | **Description**                     | **Required?** | **Type**                 |
|-----------------:|:------------------------------------|:--------------|:-------------------------|
| `targetUrl`      | URL of the link                     | Yes           | Non-empty string         |
| `elementId`      | HTML id of the link element         | No            | Non-empty string         |
| `elementClasses` | HTML classes of the link element    | No            | String                   |
| `elementTarget`  | HTML target of the link element     | No            | String                   |
| `context`        | Custom context for the event        | No            | Array                    |
| `tstamp`         | Item price                          | No            | Number                   |

[repo]: https://github.com/snowplow/snowplow-javascript-tracker/tree/master/core
[npm-page]: https://www.npmjs.org/package/snowplow-tracker-core
[server-tracker]: https://github.com/snowplow/snowplow-nodejs-tracker
[client-tracker]: https://github.com/snowplow/snowplow-javascript-tracker
[uuid-4]: http://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_.28random.29