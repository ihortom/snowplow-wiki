<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Trackers**](trackers) » Android/Java Tracker

This page refers to version 0.8.0+ of the Snowplow Java Tracker.

* [Java v0.7.*][java-0.7]
* [Java v0.5.* and Android v0.1.*][java-0.5]
* [Java v0.4.*][java-0.4]

## Contents

- 1 [Overview](#overview)
- 2 [Initialization](#init)
  - 2.1 [Importing the module](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
    - 2.2.1 [`emitter`](#emitter)
    - 2.2.2 [`subject`](#subject)
    - 2.2.3 [`namespace`](#namespace)
    - 2.2.4 [`appId`](#app-id)
    - 2.2.5 [`base64Encoded`](#base64)
    - 2.2.6 [`platform`](#platform)
- 3 [Adding extra data: the Subject class](#add-data)
  - 3.1 [`setUserId`](#set-user-id)
  - 3.2 [`setScreenResolution`](#set-screen-resolution)
  - 3.3 [`setViewport`](#set-viewport-dimensions)
  - 3.4 [`setColorDepth`](#set-color-depth)
  - 3.5 [`setTimezone`](#set-timezone)
  - 3.6 [`setLanguage`](#set-lang)
  - 3.7 [`setIpAddress`](#set-ip-address)
  - 3.8 [`setUseragent`](#set-user-agent)
  - 3.9 [`setNetworkUserId`](#set-network-user-id)
  - 3.10 [`setDomainUserId`](#set-domain-user-id)
- 4 [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Self-describing JSON](#self-describing-json)
    - 4.1.2 [Custom contexts](#custom-contexts)
    - 4.1.3 [Timestamp override](#timestamp-override)
    - 4.1.4 [Subject override](#subject-override)
    - 4.1.5 [Event ID override](#event-id-override)
  - 4.2 [`track(ScreenView event)`](#screen-view)
  - 4.3 [`track(PageView event)`](#page-view)
  - 4.4 [`track(EcommerceTransaction event)`](#ecommerce-transaction)
    - 4.4.1 [`EcommerceTransactionItem`](#ecommerce-transaction-item)
  - 4.5 [`track(Structured event)`](#struct-event)
  - 4.6 [`track(Unstructured event)`](#unstruct-event)
  - 4.7 [`track(Timing event)`](#timing)
- 5 [Sending event: `Emitter`](#emitters)
  - 5.1 [`HttpClientAdapter`](#http-client-adapters)
    - 5.1.1 [`OkHttpClientAdapter`](#ok-http-adapter)
    - 5.1.2 [`ApacheHttpClientAdapter`](#apache-adapter)
  - 5.2 [Using a buffer](#buffer)
  - 5.3 [Choosing the HTTP method](#http-method)
  - 5.4 [Thread Count](#thread-count)
  - 5.5 [Emitter callback](#http-callback)
- 6 [Payload](#payload)
  - 6.1 [TrackerPayload](#tracker-payload)
  - 6.2 [SelfDescribingJson](#selfdescribingjson)
- 7 [Logging](#logging)

<a name="overview" />

## 1. Overview

The [Snowplow Java Tracker](https://github.com/snowplow/snowplow-java-tracker) allows you to track Snowplow events from your Java-based desktop and server apps, servlets and games. It supports JDK7+.

The tracker should be straightforward to use if you are comfortable with Java development.  If you haven't already, have a look at the [[Java Tracker Setup]] or [[Android Tracker Setup]] guide before continuing.

[Back to top](#top)

<a name="init" />

## 2 Initialization

Assuming you have completed the [[Java Tracker Setup]] for your project, you are now ready to initialize the Java Tracker.

<a name="importing" />

### 2.1 Importing the module

Import the Java Tracker's classes into your Java code like so:

```java
import com.snowplowanalytics.snowplow.tracker.*;
import com.snowplowanalytics.snowplow.tracker.emitter.*;
import com.snowplowanalytics.snowplow.tracker.http.*;
```

That's it - you are now ready to initialize a Tracker instance.

[Back to top](#top)

<a name="create-tracker" />

### 2.2 Creating a Tracker

To instantiate a tracker in your code (can be global or local to the process being tracked) simply instantiate the `Tracker` interface with the following builder patterm:

```java
Tracker.TrackerBuilder(Emitter : emitter, String : namespace, String : appId)
  .subject(Subject)
  .base64(boolean)
  .platform(enum DevicePlatforms)
  .build();
```

For example:

```java
Tracker tracker = new Tracker.TrackerBuilder(emitter, "AF003", "cf")
                    .subject(user1Subject)
                    .base64(true)
                    .platform(DevicePlatform.Desktop)
                    .build();
```

| **Argument Name** | **Description**                              | **Required?**  | **Default**   |
|------------------:|:---------------------------------------------|:---------------|:--------------|
| `emitter`         | The Emitter object you create                | Yes            | Null          |
| `namespace`       | The name of the tracker instance             | Yes            | Null          |
| `appId`           | The application ID                           | Yes            | Null          |
| `subject`         | The subject that defines a user              | No             | Null          |
| `base64`          | Whether to enable [base 64 encoding][base64] | No             | True          |
| `platform`        | The device the Tracker is running on         | No             | ServerSideApp |

<a name="emitter" />

#### 2.2.1 `emitter`

The emitter to which the tracker will send events. See [Emitters](#emitter) for more on emitter configuration.

To attach a new Emitter to the Tracker:

```java
Emitter newEmitter = ...;
tracker.setEmitter(newEmitter);
```

[Back to top](#top)

<a name="subject" />

#### 2.2.2 `subject`

The user which the Tracker will track. This should be an instance of the [Subject](#subject) class. You don't need to set this during Tracker construction; you can use the `Tracker.setSubject` method afterwards. Alternatively, you can also include a Subject object with the actual event; this will override the Tracker attached subject.

In fact, you don't need to create a subject at all. If you don't, though, your events won't contain user-specific data such as timezone and language.

To attach a new Subject to the Tracker:

```java
Subject newSubject = ...;
tracker.setSubject(newSubject);
```

[Back to top](#top)

<a name="namespace" />

#### 2.2.3 `namespace`

The `namespace` argument will be attached to every event fired by the new tracker. This allows you to later identify which tracker fired which event if you have multiple trackers running.

To set a new Namespace in the Tracker:

```java
tracker.setNamespace("New-Namespace");
```

[Back to top](#top)

<a name="app-id" />

#### 2.2.4 `appId`

The `appId` argument lets you set the application ID to any string.

To set a new AppId in the Tracker:

```java
tracker.setAppId("New-App-Id");
```

[Back to top](#top)

<a name="base64" />

#### 2.2.5 `base64`

By default, unstructured events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the boolean `base64` argument.

To change whether to base64 encode:

```java
tracker.setBase64Encoded(true || false);
```

[Back to top](#top)

<a name="platform" />

### 2.2.6 Change the tracker's platform with `setPlatform`

By default this value is set to `DevicePlatform.ServerSideApp`.

You can change the platform by calling:

```java
tracker.setPlatform(DevicePlatform.{{ Enum value }});
```

For a full list of supported platforms, please see the [[Snowplow Tracker Protocol]].

[Back to top](#top)

<a name="add-data" />

## 3. Adding extra data: the Subject class

You may have additional information about your application's environment, current user and so on, which you want to send to Snowplow with each event.

The Subject class has a set of `set...()` methods to attach extra data relating to the user to all tracked events:

* [`setUserId`](#set-user-id)
* [`setScreenResolution`](#set-screen-resolution)
* [`setViewport`](#set-viewport-dimensions)
* [`setColorDepth`](#set-color-depth)
* [`setTimezone`](#set-timezone)
* [`setLanguage`](#set-lang)
* [`setIpAddress`](#set-ip-address)
* [`setUseragent`](#set-user-agent)
* [`setNetworkUserId`](#set-network-user-id)
* [`setDomainUserId`](#set-domain-user-id)

Here are some examples:

```java
// Init an empty Subject and add some values
Subject s1 = new Subject.SubjectBuilder().build();
s1.setUserId("Kevin Gleason");
s1.setLanguage("en-gb");
s1.setScreenResolution(1920, 1080);

// Init all values at build time
Subject s1 = new Subject.SubjectBuilder()
              .userId("Kevin Gleason")
              .language("en-gb")
              .screenResolution(1920, 1080)
              .build();
```

After that, you can add your Subject to your Tracker like so:

```java
Tracker t1 = new Tracker.TrackerBuilder( ... )
              .subject(s1)
              .build();

// OR

t1.setSubject(s1);
```

[Back to top](#top)

<a name="set-user-id" />

### 3.1 Set user ID with `setUserId`

You can set the user ID to any string:

```java
s1.setUserId( "{{USER ID}}" )
```

Example:

```java
s1.setUserId("alexd")
```

[Back to top](#top)

<a name="set-screen-resolution" />

### 3.2 Set screen resolution with `setScreenResolution`

If your Java code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```java
t1.setScreenResolution( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```java
t1.setScreenResolution(1366, 768)
```

[Back to top](#top)

<a name="set-viewport-dimensions" />

### 3.3 Set viewport dimensions with `setViewport`

If your Java code has access to the viewport dimensions, then you can pass this in to Snowplow too:

```java
s.setViewport( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```java
s.setViewport(300, 200)
```

[Back to top](#top)

<a name="set-color-depth" />

### 3.4 Set color depth with `setColorDepth`

If your Java code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

```java
s.setColorDepth( {{BITS PER PIXEL}} )
```

The number should be a positive integer, in bits per pixel. Example:

```java
s.setColorDepth(32)
```

[Back to top](#top)

<a name="set-timezone" />

### 3.5 Set timezone with `setTimezone`

This method lets you pass a user's timezone in to Snowplow:

```java
s.setTimezone( {{TIMEZONE}} )
```

The timezone should be a string:

```java
s.setTimezone("Europe/London")
```

[Back to top](#top)

<a name="set-lang" />

### 3.6 Set the language with `setLanguage`

This method lets you pass a user's language in to Snowplow:

```java
s.setLanguage( {{LANGUAGE}} )
```

The language should be a string:

```java
s.setLanguage('en')
```

[Back to top](#top)

<a name="set-ip-address" />

### 3.7 `setIpAddress`

This method lets you pass a user's IP Address in to Snowplow:

```java
setIpAddress( {{IP ADDRESS}} )
```

The IP address should be a string:

```java
subj.setIpAddress("127.0.0.1");
```

[Back to top](#top)

<a name="set-user-agent" />

### 3.8 `setUseragent`

This method lets you pass a useragent in to Snowplow:

```java
setUseragent( {{USERAGENT}} )
```

The useragent should be a string:

```java
subj.setUseragent("Agent Smith");
```

[Back to top](#top)

<a name="set-network-user-id" />

### 3.9 `setNetworkUserId`

This method lets you pass a Network User ID in to Snowplow:

```java
setNetworkUserId( {{NUID}} )
```

The network user id should be a string:

```java
subj.setNetworkUserId("network-id");
```

[Back to top](#top)

<a name="set-domain-user-id" />

### 3.10 `setDomainUserId`

This method lets you pass a Domain User ID in to Snowplow:

```java
setDomainUserId( {{DUID}} )
```

The domain user id should be a string:

```java
subj.setDomainUserId("domain-id");
```

[Back to top](#top)

<a name="events" />

## 4. Tracking specific events

Snowplow has been built to enable you to track a wide range of events that occur when users interact with your websites and apps. We are constantly growing the range of functions available in order to capture that data more richly.

Events supported by the Java Tracker at a glance:

| **Events**                                                    | **Description*                                         |
|--------------------------------------------------------------:|:-------------------------------------------------------|
| [`track(ScreenView event)`](#screen-view)                     | Track the user viewing a screen within the application |
| [`track(PageView event)`](#page-view)                         | Track and record views of web pages                    |
| [`track(EcommerceTransaction event)`](#ecommerce-transaction) | Track an ecommerce transaction and its items           |
| [`track(Structured event)`](#struct-event)                    | Track a Snowplow custom structured event               |
| [`track(Unstructured event)`](#unstruct-event)                | Track a Snowplow custom unstructured event             |
| [`track(Timing event)`](#timing)                              | Track a Timing with Category event                     |

You can also directly Track a `TrackerPayload` object.  **Please** only use this function to re-track failed event payloads.

```java
tracker.track(aTrackerPayload);
```

[Back to top](#top)

<a name="common" />

### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `track(XXX)`, where `XXX` is the type of event to track.

[Back to top](#top)

<a name="self-describing-json" />

#### 4.1.1 SelfDescribingJson

A `SelfDescribingJson` is used as a wrapper around either a `TrackerPayload`, another `SelfDescribingJson` or a `Map` object. After creating the object you want to wrap, you can create a `SelfDescribingJson` using the following:

```java
// This is the Map we have created
Map<String, String> eventData = new HashMap<>();
eventData.put("Event", "Data")

// We wrap that map in a SelfDescribingJson before sending it
SelfDescribingJson json = new SelfDescribingJson("iglu:com.acme/example/jsonschema/1-0-0", eventData);
```

You can create a SelfDescribingJson with the following arguments:

| **Argument** | **Description**                           | **Required?** | **Type**                 |
|-------------:|:------------------------------------------|:--------------|:-------------------------|
| `schema`     | JsonSchema that describes the data        | Yes           | `String`                 |
| `data`       | Data that will be validated by the schema | No            | `Map, TrackerPayload, SelfDescribingJson` |

`SelfDescribingJson` is used for recording [custom contexts](#custom-contexts) and [unstructured events](#unstruct-event).

[Back to top](#top)

<a name="custom-contexts" />

#### 4.1.2 Custom contexts

In short, custom contexts let you add additional information about the circumstances surrounding an event in the form of a Map object. Each tracking method accepts an additional optional contexts parameter:

```java
t1.track(PageView.builder().( ... ).customContext(List<SelfDescribingJson> context).build());
```

The `customContext` argument should consist of a `List` of `SelfDescribingJson` representing an array of one or more contexts. The format of each individual context element is the same as for an [unstructured event](#unstruct-event).

If a visitor arrives on a page advertising a movie, the context dictionary might look like this:

```json
{
  "schema": "iglu:com.acme_company/movie_poster/jsonschema/2-1-1",
  "data": {
    "movie_name": "Solaris",
    "poster_country": "JP",
    "poster_year": "1978"
  }
}
```

To construct this as a `SelfDescribingJson`:

```java
// Create a Map of the data you want to include...
Map<String, String> dataMap = new HashMap<>();
dataMap.put("movie_name", "solaris");
dataMap.put("poster_country", "JP");
dataMap.put("poster_year", "1978");

// Now create your SelfDescribingJson object...
SelfDescribingJson context1 = new SelfDescribingJson("iglu:com.acme/movie_poster/jsonschema/2.1.1", dataMap);

// Now add this JSON into a list of SelfDescribingJsons...
List<SelfDescribingJson> contexts = new ArrayList<>();
contexts.add(context1);
```

Note that even if there is only one custom context attached to the event, it still needs to be placed in an array.

[Back to top](#top)

<a name="timestamp-override" />

### 4.1.3 Optional Timestamp override

In all the trackers, we offer a way to override the timestamp if you want the event to show as tracked at a specific time. If you don't, we create a timestamp while the event is being tracked.

Here is an example:

```java
t1.track(PageView.builder().( ... ).timestamp(1423583655000).build());
```

[Back to top](#top)

<a name="subject-override" />

### 4.1.4 Optional Subject override

In this tracker, we offer a way to attach an event specific `Subject` if you want the event to be specific to a certain user. If you don't, we attempt to attach the Tracker subject or no Subject if neither are available.

This is useful for quickly overriding the Tracker subject for specific users.

Here is an example:

```java
t1.track(PageView.builder().( ... ).subject(aUserSubject).build());
```

[Back to top](#top)

<a name="event-id-override" />

### 4.1.5 Optional Event ID override

In this tracker, we offer a way to override the event id of a specific event instead of using the automatically generated one. If you don't use this option we will simply use the auto-generated alternative.

Here is an example:

```java
t1.track(PageView.builder().( ... ).eventId("custom-event-id").build());
```

[Back to top](#top)

<a name="screen-view" />

#### 4.2 Track screen views with `track(ScreenView event)`

Use `track(ScreenView event)` to track a user viewing a screen (or equivalent) within your app. You must use either `name` or `id`. Arguments are:

| **Argument**   | **Description**                     | **Required?** | **Type**                   |
|---------------:|:------------------------------------|:--------------|:---------------------------|
| `name`         | Human-readable name for this screen | No            | `String`                   |
| `id`           | Unique identifier for this screen   | No            | `String`                   |
| `customContext`| Optional custom context             | No            | `List<SelfDescribingJson>` |
| `timestamp`    | Optional timestamp                  | No            | `Long`                     |
| `eventId`      | Optional custom event id            | No            | `String`                   |
| `subject`      | Optional custom Subject object      | No            | `Subject`                  |

Examples:

```java
t1.track(ScreenView.builder()
    .name("HUD > Save Game")
    .id("screen23")
    .build());

t1.track(ScreenView.builder()
    .name("HUD > Save Game")
    .id("screen23")
    .customContext(contextList)
    .timestamp(1423583655000)
    .eventId("uid-1")
    .build());
```

[Back to top](#top)

<a name="page-view" />

#### 4.3 Track pageviews with `track(PageView event)`

You can use `track(PageView event)` to track a user viewing a web page within your app.

Arguments are:

| **Argument**   | **Description**                      | **Required?** | **Type**                   |
|---------------:|:-------------------------------------|:--------------|:---------------------------|
| `pageUrl`      | The URL of the page                  | Yes           | `String`                   |
| `pageTitle`    | The title of the page                | No            | `String`                   |
| `referrer`     | The address which linked to the page | No            | `String`                   |
| `customContext`| Optional custom context              | No            | `List<SelfDescribingJson>` |
| `timestamp`    | Optional timestamp                   | No            | `Long`                     |
| `eventId`      | Optional custom event id             | No            | `String`                   |
| `subject`      | Optional custom Subject object       | No            | `Subject`                  |

Examples:

```java
t1.track(PageView.builder()
    .pageUrl("www.example.com")
    .pageTitle("example")
    .referrer("www.referrer.com")
    .build());

t1.track(PageView.builder()
    .pageUrl("www.example.com")
    .pageTitle("example")
    .referrer("www.referrer.com")
    .customContext(contextList)
    .timestamp(1423583655000)
    .eventId("uid-1")
    .build());
```

[Back to top](#top)

<a name="ecommerce-transaction" />

#### 4.4 Track ecommerce transactions with `track(EcommerceTransaction event)`

Use `track(EcommerceTransaction event)` to track an ecommerce transaction.

Arguments:

| **Argument**   | **Description**                      | **Required?** | **Type**                   |
|---------------:|:-------------------------------------|:--------------|:---------------------------|
| `orderId`      | ID of the eCommerce transaction      | Yes           | `String`                   |
| `totalValue`   | Total transaction value              | Yes           | `Double`                   |
| `affiliation`  | Transaction affiliation              | No            | `String`                   |
| `taxValue`     | Transaction tax value                | No            | `Double`                   |
| `shipping`     | Delivery cost charged                | No            | `Double`                   |
| `city`         | Delivery address city                | No            | `String`                   |
| `state`        | Delivery address state               | No            | `String`                   |
| `country`      | Delivery address country             | No            | `String`                   |
| `currency`     | Transaction currency                 | No            | `String`                   |
| `items`        | Items in the transaction             | Yes           | `List<EcommerceTransactionItem>`    |
| `items`        | Items in the transaction             | Yes           | `EcommerceTransactionItem...` |
| `customContext`| Optional custom context              | No            | `List<SelfDescribingJson>` |
| `timestamp`    | Optional timestamp                   | No            | `Long`                     |
| `eventId`      | Optional custom event id             | No            | `String`                   |
| `subject`      | Optional custom Subject object       | No            | `Subject`                  |

The `items` argument is a `List` of individual `EcommerceTransactionItem` elements representing the items in the e-commerce transaction or it can be a `varargs` argument of many individual items. Note that `track(EcommerceTransaction event)` fires multiple events: one transaction event for the transaction as a whole, and one transaction item event for each element of the `items` `List`. Each transaction item event will have the same timestamp, order_id, and currency as the main transaction event.

[Back to top](#top)

<a name="ecommerce-transaction-item" />

#### 4.4.1 `EcommerceTransactionItem`

To instantiate a `EcommerceTransactionItem` in your code, simply use the following constructor signature:

```java
EcommerceTransactionItem item = EcommerceTransactionItem.builder()
    .itemId("item_id")
    .sku("item_sku")
    .price(1.00)
    .quantity(1)
    .name("item_name")
    .category("item_category")
    .currency("currency")
    .build();
```

These are the fields that can appear as elements in each `EcommerceTransactionItem` element of the transaction item's `List`:

| **Field**      | **Description**                      | **Required?** | **Type**                   |
|---------------:|:-------------------------------------|:--------------|:---------------------------|
| `itemId`       | Item ID                              | Yes           | `String`                   |
| `sku`          | Item SKU                             | Yes           | `String`                   |
| `price`        | Item price                           | Yes           | `Double`                   |
| `quantity`     | Item quantity                        | Yes           | `Integer`                  |
| `name`         | Item name                            | No            | `String`                   |
| `category`     | Item category                        | No            | `String`                   |
| `currency`     | Item currency                        | No            | `String`                   |
| `customContext`| Optional custom context              | No            | `List<SelfDescribingJson>` |
| `timestamp`    | Optional timestamp                   | No            | `Long`                     |
| `eventId`      | Optional custom event id             | No            | `String`                   |
| `subject`      | Optional custom Subject object       | No            | `Subject`                  |

Example of tracking a transaction containing two items:

```java
// Create some Transaction Items
EcommerceTransactionItem item1 = EcommerceTransactionItem.builder()
    .itemId("item_id_1")
    .sku("item_sku_1")
    .price(1.00)
    .quantity(1)
    .name("item_name")
    .category("item_category")
    .currency("currency")
    .build();

EcommerceTransactionItem item2 = EcommerceTransactionItem.builder()
    .itemId("item_id_2")
    .sku("item_sku_2")
    .price(1.00)
    .quantity(1)
    .name("item_name")
    .category("item_category")
    .currency("currency")
    .build();

// Add these items to a List
List<EcommerceTransactionItem> items = new ArrayList<>();
items.add(item1);
items.add(item2);

// Now Track the Transaction by using this list of items as an argument
tracker.track(EcommerceTransaction.builder()
    .orderId("6a8078be")
    .totalValue(300.00)
    .affiliation("my_affiliate")
    .taxValue(30.00)
    .shipping(10.00)
    .city("Boston")
    .state("Massachusetts")
    .country("USA")
    .currency("USD")
    .items(items)
    .build());

// Or include the items as varargs in the items section
tracker.track(EcommerceTransaction.builder()
    .orderId("6a8078be")
    .totalValue(300.00)
    .affiliation("my_affiliate")
    .taxValue(30.00)
    .shipping(10.00)
    .city("Boston")
    .state("Massachusetts")
    .country("USA")
    .currency("USD")
    .items(item1, item2)
    .build());
```

[Back to top](#top)

<a name="struct-event" />

#### 4.5 Track structured events with `track(Structured event)`

Use `track(Structured event)` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument**   | **Description**                                                  | **Required?** | **Type**                   |
|---------------:|:---------------------------------------------------------------  |:--------------|:---------------------------|
| `category`     | The grouping of structured events which this `action` belongs to | Yes           | `String`                   |
| `action`       | Defines the type of user interaction which this event involves   | Yes           | `String`                   |
| `label`        | A string to provide additional dimensions to the event data      | No            | `String`                   |
| `property`     | A string describing the object or the action performed on it     | No            | `String`                   |
| `value`        | A value to provide numerical data about the event                | No            | `Double`                   |
| `customContext`| Optional custom context                                          | No            | `List<SelfDescribingJson>` |
| `timestamp`    | Optional timestamp                                               | No            | `Long`                     |
| `eventId`      | Optional custom event id                                         | No            | `String`                   |
| `subject`      | Optional custom Subject object                                   | No            | `Subject`                  |

Examples:

```java
t1.track(Structured.builder()
    .category("shop")
    .action("add-to-basket")
    .label("Add To Basket")
    .property("pcs")
    .value(2.00)
    .build());

t1.track(Structured.builder()
    .category("shop")
    .action("add-to-basket")
    .label("Add To Basket")
    .property("pcs")
    .value(2.00)
    .customContext(contextList)
    .timestamp(1423583655000)
    .eventId("uid-1")
    .build());
```

[Back to top](#top)

<a name="unstruct-event" />

#### 4.6 Track unstructured events with `track(Unstructured event)`

Custom unstructured events are a flexible tool that enable Snowplow users to define their own event types and send them into Snowplow.

When a user sends in a custom unstructured event, they do so as a JSON of name-value properties, that conforms to a JSON schema defined for the event earlier.

Use `track(Unstructured event)` to track a custom event which consists of a name and an unstructured set of properties. This is useful when:

* You want to track event types which are proprietary/specific to your business (i.e. not already part of Snowplow), or
* You want to track events which have unpredictable or frequently changing properties

The arguments are as follows:

| **Argument**   | **Description**                   |  **Required?** |  **Type**                  |
|---------------:|:----------------------------------|:---------------|:---------------------------|
| `eventData`    | The properties of the event       | Yes            | `SelfDescribingJson`       |
| `customContext`| Optional custom context           | No             | `List<SelfDescribingJson>` |
| `timestamp`    | Optional timestamp                | No             | `Long`                     |
| `eventId`      | Optional custom event id          | No             | `String`                   |

Example event json to track:

```json
{
  "schema": "iglu:com.acme/save_game/jsonschema/1-0-0",
  "data": {
    "levelName": "Barrels o' Fun",
    "levelIndex": 23
  }
}
```

How to set it up?

```java
// Create a Map of your event data
Map<String, Object> eventMap = new HashMap<>();
eventMap.put("levelName", "Barrels o' Fun")
eventMap.put("levelIndex", 23);

// Create your event data
SelfDescribingJson eventData = new SelfDescribingJson("iglu:com.acme/save_game/jsonschema/1-0-0", eventMap);

// Track your event with your custom event data
t1.track(Unstructured.builder()
    .eventData(eventData)
    .build();

// OR

t1.track(Unstructured.builder()
    .eventData(eventData)
    .customContext(contextList)
    .timestamp(1423583655000)
    .eventId("uid-1")
    .build();
```

For more on JSON schema, see the [blog post][self-describing-jsons].

[Back to top](#top)

<a name="timing" />

### 4.7 Track timing events with `track(Timing event)`

Use `track(Timing event)` to track an event related to a custom timing.

| **Argument**   | **Description**                                 | **Required?** | **Type**                   |
|---------------:|:------------------------------------------------|:--------------|:---------------------------|
| `category`     | The category of the timed event                 | Yes           | `String`                   |
| `label`        | The label of the timed event                    | No            | `String`                   |
| `timing`       | The timing measurement in milliseconds          | Yes           | `Integer`                  |
| `variable`     | The name of the timed event                     | Yes           | `String`                   |
| `customContext`| Optional custom context                         | No            | `List<SelfDescribingJson>` |
| `timestamp`    | Optional timestamp                              | No            | `Long`                     |
| `eventId`      | Optional custom event id                        | No            | `String`                   |

Examples:

```java
t1.track(Timing.builder()
    .category("category")
    .variable("variable")
    .timing(1)
    .label("label")
    .build());

t1.track(Timing.builder()
    .category("category")
    .variable("variable")
    .timing(1)
    .label("label")
    .customContext(contextList)
    .timestamp(1423583655000)
    .eventId("uid-1")
    .build());
```

[Back to top](#top)

<a name="emitters" />

## 5. Sending event: `Emitter`

Events are sent using an `Emitter` class. You can initialize a class using a variety of builder functions.

Here are the Emitter builder functions that can be used to make either a `SimpleEmitter` or `BatchEmitter`:

```java
// Simple (GET) Emitter
Emitter simple = SimpleEmitter.builder()
        .httpClientAdapter( {{ An Adapter }} ) // Required
        .threadCount(20) // Default is 50
        .requestCallback( {{ A callback }} ) // Default is Null
        .build();

// Batch (POST) Emitter
Emitter batch = BatchEmitter.builder()
        .httpClientAdapter( {{ An Adapter }} ) // Required
        .bufferSize(20)  // Default is 50
        .threadCount(20) // Default is 50
        .requestCallback( {{ A callback }} ) // Default is Null
        .build();
```

| **Function Name**  | **Description**                                                             |    **Required?**  |
|-------------------:|:----------------------------------------------------------------------------|:------------------|
| `httpClientAdapter`| The `HttpClientAdapter` to use for all event sending                        | Yes               |
| `bufferSize`       | BatchEmitter Only: Specifies how many events go into a POST                 | No                |
| `threadCount`      | The count of Threads that can be used to send events                        | No                |
| `requestCallback`  | Lets you pass a callback class to handle succes/failure in sending events   | No                |


[Back to top](#top)

<a name="http-client-adapters" />

### 5.1 `HttpClientAdapters`

We currently offer two different Http Clients that can be used to send events to our collectors.  Once created they need to be attached to the emitter in the `httpClientAdapter` builder argument.

| **Function Name**  | **Description**                                                             |    **Required?**  |
|-------------------:|:----------------------------------------------------------------------------|:------------------|
| `url`              | The URL of the collector to send events to                                  | Yes               |
| `httpClient`       | The http client to use (either OkHttp or Apache)                            | Yes               |

[Back to top](#top)

<a name="ok-http-adapter" />

### 5.1.1 `OkHttpClientAdapter`

You build an `OkHttpClientAdapter` like so:

```java
// Make a new client
OkHttpClient client = new OkHttpClient();

// Optionally configure the client
client.setConnectTimeout(5, TimeUnit.SECONDS);
client.setReadTimeout(5, TimeUnit.SECONDS);
client.setWriteTimeout(5, TimeUnit.SECONDS);

// Build the adapter
HttpClientAdapter adapter = OkHttpClientAdapter.builder()
      .url("http://www.acme.com")
      .httpClient(client)
      .build();
```

[Back to top](#top)

<a name="apache-adapter" />

### 5.1.2 `ApacheHttpClientAdapter`

You build an `ApacheHttpClientAdapter` like so:

```java
// Make a new client with custom concurrency rules
PoolingHttpClientConnectionManager manager = new PoolingHttpClientConnectionManager();
manager.setDefaultMaxPerRoute(50);

// Make the client
CloseableHttpClient client = HttpClients.custom()
      .setConnectionManager(manager)
      .build();

// Build the adapter
HttpClientAdapter adapter = ApacheHttpClientAdapter.builder()
      .url("http://www.acme.com")
      .httpClient(client)
      .build();
```

**NOTE**: it is encouraged to research how best you want to setup your `ApacheClient` for maximum performance.  By default the Apache Client will never timeout and will also allow only two outbound connections at a time.  We have used a `PoolingHttpClientConnectionManager` here to override that setting to allow up to 50 concurrent outbound connections.

[Back to top](#top)

<a name="buffer" />

### 5.2 Using a buffer

**NOTE**: Only applies to the `BatchEmitter`

A buffer is used to group events together in bulk before sending them. This is especially handy to reduce network usage. By default, the Emitter buffers up to 50 events before sending them. You can change this to send events instantly as soon as they are created like so:

```java
Emitter batch = BatchEmitter.builder()
        .httpClientAdapter( ... )
        .build();

batch.setBufferSize(1)
```

The buffer size must be an integer greater than or equal to 1.

[Back to top](#top)

<a name="http-method" />

###  5.3 Choosing the HTTP method

Choosing to send via GET or POST is as easy as building a particular type of Emitter.  If you want to send via GET then you will need to build a `SimpleEmitter`.  For sending via POST you will need to build a `BatchEmitter`.

[Back to top](#top)

<a name="thread-count" />

###  5.4 Thread Count

The Thread Count is the size of the available Thread Pool that events can be sent on.  The bigger the Pool of Threads the faster events can be sent.  By default we use 50 Threads for sending but this can be altered up or down depending on many events you are sending.  As well as how strong a computer the Tracker is running on.

[Back to top](#top)

<a name="http-callback" />

###  5.5 Emitter callback

If an event fails to send because of a network issue, you can choose to handle the failure case with a callback class to react accordingly. The callback class needs to implement the `RequestCallback` interface in order to do so. Here is a sample bit of code to show how it could work:

```java
// Make a RequestCallback
RequestCallback callback = new RequestCallback() {
  @Override
  public void onSuccess(int successCount) {
    System.out.println("Success, successCount: " + successCount);
  }

  @Override
  public void onFailure(int successCount, List<TrackerPayload> failedEvents) {
    System.out.println("Failure, successCount: " + successCount + "\nfailedEvent:\n" + failedEvents.toString());
  }
};

// Attach it to an Emitter
Emitter e1 = BatchEmitter.builder()
        .httpClientAdapter( ... )
        .requestCallback(callback)
        .build();
```

In the example, we can see an in-line example of handling the case. If events are all successfully sent, the `onSuccess` method returns the number of successful events sent. If there were any failures, the `onFailure` method returns the successful events sent (if any) and a *list of events* that failed to be sent (i.e. the HTTP state code did not return 200).

A common pattern here could be to re-send all failed events if they occur.  It is up to the developer to determine whether they want to wait a certain amount of time before re-sending or if they want to re-send at all.

```java
// Example on-failure function with re-tracking
RequestCallback callback = new RequestCallback() {
  @Override
  public void onSuccess(int successCount) {
    System.out.println("Success, successCount: " + successCount);
  }

  @Override
  public void onFailure(int successCount, List<TrackerPayload> failedEvents) {
    System.out.println("Failure, successCount: " + successCount + "\nfailedEvent:\n" + failedEvents.toString());

    // Re-send each event
    for (TrackerPayload payload : failedEvents) {
      tracker.tracker(payload);
    }
  }
};
```

[Back to top](#top)

<a name="payload" />

## 6. Payload

A Payload interface is used for implementing a [TrackerPayload](#tracker-payload) and [SelfDescribingJson](#selfdescribingjson), but accordingly, can be used to implement your own Payload class if you choose.

[Back to top](#top)

<a name="tracker-payload" />

### 6.1 Tracker Payload

A TrackerPayload is used internally within the Java Tracker to create the tracking event payloads that are passed to an Emitter to be sent accordingly. It is essentially a wrapper around a `LinkedHashMap<String, String>` and does basic validation to ensure all key-value pairs are valid non-null and non-empty Strings.

[Back to top](#top)

<a name="selfdescribingjson" />

### 6.2 SelfDescribingJson

A SelfDescribingJson is used primarily to ease construction of self-describing JSON objects. It is a wrapper around a `LinkedHashMap<String, Object>` and will only ever contain two key-value pairs.  A `schema` key with a valid schema value and a `data` key containing a `Map` of key-value pairs.

This is used under the hood but is also useful for to know about when attaching custom contexts to events or creating `Unstructured` events.

Here's a short example:

```java
// This is the Map we have created
Map<String, String> eventData = new HashMap<>();
eventData.put("Event", "Data")

// We wrap that map in a SelfDescribingJson before sending it
SelfDescribingJson json = new SelfDescribingJson("iglu:com.acme/example/jsonschema/1-0-0", eventData);
```

[Back to top](#top)

<a name="logging" />

## 7. Logging

Logging in the Tracker is done using SLF4J. The majority of the logging set as `DEBUG` so it will not overly populate your own logging.

[Back to top](#top)

[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[java-0.7]: https://github.com/snowplow/snowplow/wiki/Android-and-Java-Tracker
[java-0.5]: https://github.com/snowplow/snowplow/wiki/Android-v0.1-and-Java-Tracker-v0.5
[java-0.4]: https://github.com/snowplow/snowplow/wiki/Java-Tracker-v0.4
[base64]: https://en.wikipedia.org/wiki/Base64
