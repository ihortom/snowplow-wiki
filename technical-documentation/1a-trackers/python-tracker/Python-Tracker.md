<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Trackers**](trackers) » Python Tracker

*This page refers to version 0.8.0 of the Snowplow Python Tracker, which is the latest version. Documentation for earlier versions is available:*

*[Version 0.2][python-0.2]*
*[Version 0.3][python-0.3]*
*[Version 0.4][python-0.4]*
*[Version 0.5][python-0.5]*
*[Version 0.6][python-0.6]*
*[Version 0.7][python-0.7]*

## Contents

- 1 [Overview](#overview)
- 2 [Initialization](#init)
  - 2.1 [Importing the module](#importing)
  - 2.2 [Creating a tracker](#create-tracker)
    - 2.2.1 [`emitters`](#emitter)
    - 2.2.2 [`subject`](#subject)
    - 2.2.3 [`namespace`](#namespace)
    - 2.2.4 [`app_id`](#app-id)
    - 2.2.5 [`encode_base64`](#base64)
- 3 [Adding extra data: the Subject class](#subject-class)
  - 3.1 [`set_platform`](#set-platform)
  - 3.2 [`set_user_id`](#set-user-id)
  - 3.3 [`set_screen_resolution`](#set-screen-resolution)
  - 3.4 [`set_viewport`](#set-viewport-dimensions)
  - 3.5 [`set_color_depth`](#set-color-depth)
  - 3.6 [`set_timezone`](#set-timezone)
  - 3.7 [`set_lang`](#set-lang)
  - 3.8 [`set_ip_address`](#set-ip-address)
  - 3.9 [`set_useragent`](#set-useragent)
  - 3.10 [`set_domain_user_id`](#set-domain-user-id)
  - 3.11 [`set_network_user_id`](#set-network-user-id)
- 4 [Tracking specific events](#events)
  - 4.1 [Common](#common)
    - 4.1.1 [Custom contexts](#custom-contexts)
    - 4.1.2 [Optional timestamp argument](#tstamp-arg)
    - 4.1.3 [Tracker method return values](#return-values)
  - 4.2 [`track_self_describing_event()`](#selfdesc-alias)
  - 4.3 [`track_page_view()`](#page-view)
  - 4.4 [`track_page_ping()`](#page-ping)
  - 4.5 [`track_screen_view()`](#screen-view)
  - 4.6 [`track_ecommerce_transaction()`](#ecommerce-transaction)
  - 4.7 [`track_ecommerce_transaction_item()`](#ecommerce-transaction-item)
  - 4.8 [`track_struct_event()`](#struct-event)
  - 4.9 [`track_link_click()`](#link-click)
  - 4.10 [`track_add_to_cart()`](#add-to-cart)
  - 4.11 [`track_remove_from_cart()`](#remove-from-cart)
  - 4.12 [`track_form_change()`](#form-change)
  - 4.13 [`track_form_submit()`](#form-submit)
  - 4.14 [`track_site_search()`](#site-search)
  - 4.15 [`track_unstruct_event()`](#unstruct-event)
- 5 [Emitters](#emitters)
  - 5.1 [The basic Emitter class](#base-emitter)
  - 5.2 [The AsyncEmitter class](#async-emitter)
  - 5.3 [The CeleryEmitter class](#celery-emitter)
  - 5.4 [The RedisEmitter class](#redis-emitter)
  - 5.5 [Manual flushing](#manual-flushing)
  - 5.6 [Multiple emitters](#multiple-emitters)
  - 5.7 [Custom emitters](#custom-emitters)
  - 5.8 [Automatically retry sending failed events](#onfailure-loop)
  - 5.9 [Setting flush timer](#set-flush-timer)
- 6 [Contracts](#contracts)
- 7 [Logging](#logging)
- 8 [The RedisWorker class](#redis-worker)


<a name="overview" />

## 1. Overview

The [Snowplow Python Tracker](https://github.com/snowplow/snowplow-python-tracker) allows you to track Snowplow events from your Python apps and games.

The tracker should be straightforward to use if you are comfortable with Python development; any prior experience with Snowplow's [[JavaScript Tracker]] or [[Lua Tracker]], Google Analytics or Mixpanel (which have similar APIs to Snowplow) is helpful but not necessary.

Note that this tracker has access to a more restricted set of Snowplow events than the [[JavaScript Tracker]] and covers almost all the events from the [[Lua Tracker]].

There are three basic types of object you will create when using the Snowplow Python Tracker: subjects, emitters, and trackers.

A subject represents a user whose events are tracked. A tracker constructs events and sends them to one or more emitters. Each emitter then sends the event to the endpoint you configure. This will usually be a Snowplow collector, but could also be a Redis database or Celery task queue.

**A note on compatibility**

Version 0.6.0 of the Python Tracker sends POST requests and custom contexts in a format which earlier versions of Snowplow don't accept. If you wish to send POST requests or custom contexts, you must be using Snowplow version 0.9.14 or later.

<a name="init" />

## 2 Initialization

Assuming you have completed the [[Python Tracker Setup]] for your Python project, you are now ready to initialize the Python Tracker.

<a name="importing" />

### 2.1 Importing the module

Require the Python Tracker's module into your Python code like so:

```python
from snowplow_tracker import Subject, Tracker, Emitter
```

That's it - you are now ready to initialize a tracker instance.

[Back to top](#top)

<a name="create-tracker" />

### 2.2 Creating a tracker

The simplest tracker initialization only requires you to provide the URI of the collector to which the tracker will log events:

```python
e = Emitter("d3rkrsqld9gmqf.cloudfront.net")
t = Tracker(e)
```

There are other optional keyword arguments:

| **Argument Name** | **Description**                               | **Required?** | **Default**         |
|------------------:|:----------------------------------------------|:--------------|:--------------------|
| `emitters`        | The emitter to which events are sent          | Yes           | `None`              |
| `subject`         | The user being tracked                        | No            | `subject.Subject()` |
| `namespace`       | The name of the tracker instance              | No            | `None`              |
| `app_id`          | The application ID                            | No            | `None`              |
| `encode_base64`   | Whether to enable [base 64 encoding][base64] | No            | `True`              |

A more complete example:

```python
tracker = Tracker( Emitter("d3rkrsqld9gmqf.cloudfront.net") , namespace="cf", app_id="cf29ea", encode_base64=False)
```

[Back to top](#top)

<a name="emitter" />

#### 2.2.1 `emitters`

This can be a single emitter or an array containing at least one emitter. The tracker will send events to these emitters, which will in turn send them to a collector. See [Emitters](#emitters) for more on emitter configuration.

<a name="subject" />

#### 2.2.2 `subject`

The user which the Tracker will track. This should be an instance of the [Subject](#subject) class. You don't need to set this during Tracker construction; you can use the `Tracker.set_subject` method afterwards. In fact, you don't need to create a subject at all. If you don't, though, your events won't contain user-specific data such as timezone and language.

<a name="namespace" />

#### 2.2.3 `namespace`

If provided, the `namespace` argument will be attached to every event fired by the new tracker. This allows you to later identify which tracker fired which event if you have multiple trackers running.

<a name="app-id" />

#### 2.2.4 `app_id`

The `app_id` argument lets you set the application ID to any string.

<a name="base64" />

#### 2.2.5 `encode_base64`

By default, unstructured events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the Boolean `encode_base64` argument.

[Back to top](#top)

<a name="subject-class" />

## 3. Adding extra data: The Subject class

You may have additional information about the user (i.e. subject) performing the action or the environment in which the user has performed the action, some of that additional data can be sent into Snowplow with each event as part of the subject class.

Create a subject like this:

```python
from snowplow_tracker import Subject
s = Subject()
```

The Subject class has a set of `set...()` methods to attach extra data relating to the user to all tracked events:

* [`set_platform`](#set-platform)
* [`set_user_id`](#set-user-id)
* [`set_screen_resolution`](#set-screen-resolution)
* [`set_viewport`](#set-viewport-dimensions)
* [`set_color_depth`](#set-color-depth)
* [`set_timezone`](#set-timezone)
* [`set_lang`](#set-lang)

If you initialize a `Tracker` instance without a subject, a default `Subject` instance will be attached to the tracker. You can access that subject like this:

```python
t = Tracker(my_emitter)
t.subject.set_platform("mob").set_user_id("user-12345").set_lang("en")
```

We will discuss each of these in turn below:

<a name="set-platform" />

#### 3.1 Change the tracker's platform with `set_platform`

The default platform is "pc". You can change the platform the subject is using by calling:

```python
s.set_platform( {{PLATFORM}} )
```

For example:

```python
s.set_platform("tv") # Running on a Connected TV
```

For a full list of supported platforms, please see the [Snowplow Tracker Protocol](Snowplow-Tracker-Protocol#1-common-parameters-platform-and-event-independent).

[Back to top](#top)

<a name="set-user-id" />

### 3.2 Set user ID with `set_user_id`

You can set the user ID to any string:

```python
s.set_user_id( "{{USER ID}}" )
```

Example:

```python
s.set_user_id("alexd")
```

[Back to top](#top)

<a name="set-screen-resolution" />

### 3.3 Set screen resolution with `set_screen_resolution`

If your Python code has access to the device's screen resolution, then you can pass this in to Snowplow too:

```python
s.set_screen_resolution( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```python
s.set_screen_resolution(1366, 768)
```

[Back to top](#top)

<a name="set-viewport-dimensions" />

### 3.4 Set viewport dimensions with `set_viewport`

If your Python code has access to the viewport dimensions, then you can pass this in to Snowplow too:

```python
s.set_viewport( {{WIDTH}}, {{HEIGHT}} )
```

Both numbers should be positive integers; note the order is width followed by height. Example:

```python
s.set_viewport(300, 200)
```

[Back to top](#top)

<a name="set-color-depth" />

### 3.5 Set color depth with `set_color_depth`

If your Python code has access to the bit depth of the device's color palette for displaying images, then you can pass this in to Snowplow too:

```python
s.set_color_depth( {{BITS PER PIXEL}} )
```

The number should be a positive integer, in bits per pixel. Example:

```python
s.set_color_depth(32)
```

[Back to top](#top)

<a name="set-timezone" />

### 3.6 Set timezone with `set_timezone`

This method lets you pass a user's timezone into Snowplow:

```python
s.set_timezone( {{TIMEZONE}} )
```

The timezone should be a string:

```python
s.set_timezone("Europe/London")
```

[Back to top](#top)

<a name="set-lang" />

### 3.7 Set the language with `set_lang`

This method lets you pass a user's language into Snowplow:

```python
s.set_lang( {{LANGUAGE}} )
```

The language should be a string:

```python
s.set_lang('en')
```

<a name="set-ip-address" />

### 3.8 Setting the IP address with `set_ip_address`

If you have access to the user's IP address, you can set it like this:

```python
s.set_ip_address('34.633.11.139')
```

<a name="set-useragent" />

### 3.9 Setting the useragent with `set_useragent`

If you have access to the user's useragent (sometimes called "browser string"), you can set it like this:

```python
s.set_useragent('Mozilla/5.0 (Windows NT 5.1; rv:23.0) Gecko/20100101 Firefox/23.0')
```

<a name="set-domain-user-id" />

### 3.10 Setting the domain user ID with `set_domain_user_id`

The `domain_userid` field of the Snowplow event model corresponds to the ID stored in the first party cookie set by the Snowplow JavaScript Tracker. If you want to match up server-side events with client-side events, you can set the domain user ID for server-side events like this:

```python
s.set_domain_user_id('c7aadf5c60a5dff9')
```

You can extract the domain user ID from the cookies of a request using the `get_domain_user_id` function below.
The `request` argument is the [Django request object][django-request].

**Note that this function has not been tested.**

```python
import re
def snowplow_cookie(request):
    for name in request.COOKIES:
        if re.match(r"_sp_id", name) != None:
           return request.COOKIES[name]
    return None

def get_domain_user_id(request):
    cookie = snowplow_cookie(request)
    if cookie != None:
        return cookie.split(".")[0]
```

If you used the "cookieName" configuration option of the Snowplow JavaScript Tracker, replace "_sp_" with the same string you passed as the cookieName.

<a name="set-network-user-id" />

### 3.11 Setting the network user ID with `set_network_user_id`

The `network_user_id` field of the Snowplow event model corresponds to the ID stored in the third party cookie set by the Snowplow Clojure Collector. You can set the network user ID for server-side events like this:

```python
s.set_network_user_id('ecdff4d0-9175-40ac-a8bb-325c49733607')
```

[Back to top](#top)

<a name="multiple-subjects" />

### 3.7 Tracking multiple subjects

You may want to track more than one subject concurrently. To avoid data about one subject being added to events pertaining to another subject, create two subject instances and switch between them using `Tracker.set_subject`:

```python
from snowplow_tracker import Subject, Emitter, Tracker

# Create a simple Emitter which will log events to http://d3rkrsqld9gmqf.cloudfront.net/i
e = Emitter("d3rkrsqld9gmqf.cloudfront.net")

# Create a Tracker instance
t = Tracker(emitters=e, namespace="cf", app_id="CF63A")

# Create a Subject corresponding to a pc user
s1 = Subject()

# Set some data for that user
s1.set_platform("pc")
s1.set_user_id("0a78f2867de")

# Set s1 as the tracker subject
# All events fired will have the information we set about s1 attached
t.set_subject(s1)

# Track user s1 viewing a page
t.track_page_view("http://www.example.com")

# Create another Subject instance corresponding to a mobile user
s2 = Subject()

# All methods of the Subject class return the Subject instance so methods can be chained:
s2.set_platform("mob").set_user_id("0b08f8be3f1")

# Change the tracker subject from s1 to s2
# All events fired will have instead have information we set about s2 attached
t.set_subject(s2)

# Track user s2 viewing a page
t.track_page_view("http://www.example.com")

# Switch back to s1 and track a structured event, this time using method chaining:
t.set_subject(s1).track_struct_event("Ecomm", "add-to-basket", "dog-skateboarding-video", "hd", 13.99)
```

<a name="events" />

## 4. Tracking specific events

As a Snowplow user, you have the ability to define your own event types, upload the associated schemas for those types to your own Iglu schema registry and then track those events in Snowplow using the `track_self_describing_event()` method.

In addition, Snowplow has a wide selection of pre-defined events and associated and associated methods for tracking:

| **Function**                                               | **Description**                                        |
|-----------------------------------------------------------:|:-------------------------------------------------------|
| [`track_page_view()`](#page-view)                          | Track views of web pages                               |
| [`track_page_ping()`](#page-ping)                          | Track engagement on web pages over time                |
| [`track_screen_view()`](#screen-view)                      | Track screen views (non-web e.g. in-app)               |
| [`track_ecommerce_transaction()`](#ecommerce-transaction)  | Track ecommerce transaction                            |
| [`track_struct_event()`](#struct-event)                    | Track a Snowplow custom structured event               |


<a name="common" />

### 4.1 Common

All events are tracked with specific methods on the tracker instance, of the form `track_XXX()`, where `XXX` is the name of the event to track.

<a name="custom-contexts" />

### 4.1.1 Custom contexts

When you track an event, some of the data that you track will be specific to that event. A lot of the data you want to record, however, will describe entities that are tracked across multiple events. For example, a media company might want to track the following events:

* User views video listing
* User plays video
* User pauses video
* User shares video
* User favorites video
* User reviews video

Whilst each of those events is a different type, all of them involve capturing data about the user and the video. Both the 'user' and 'video' are entities that are tracked across multiple event types. Both are candidates to be "custom context". You as a Snowplow user can define your own custom contexts (including associated schemas) and then send data for as many custom contexts as you wish with *any* Snowplow event. So if you want, you can define your own "user context", and then send additional user data in that object with any event. Other examples of context include:

* articles
* videos
* products
* categories
* pages / page_types
* environments

Each tracking method accepts an additional optional contexts parameter after all the parameters specific to that method:

```python
def track_page_view(self, page_url, page_title=None, referrer=None, context=None, tstamp=None):
```

The `context` argument should consist of an array of one or more instances of `SelfDescribingJson` class.
This class isomorphic to [self-describing JSON](#selfdesc-alias), to be more precisely - it has Iglu URI attribute and data itself.

If server-side Python application can determine visitor's geoposition - it can be attached to the event, using following context (`geolocation_context` is predefined on [Iglu Central][iglu-central]):

```python
from snowplow_tracker import SelfDescribingJson

geo_context = SelfDescribingJson(
  "iglu:com.snowplowanalytics.snowplow/geolocation_context/jsonschema/1-0-0",
  {
    "latitude": -23.2,
    "longitude": 43.0
  }
)
```

If a visitor arrives on a page advertising a movie, the context object might look like this (`movie_poster` is custom context):


```python
poster_context = SelfDescribingJson(
  "iglu:com.acme_company/movie_poster/jsonschema/2-1-1",
  {
    "movie_name": "Solaris",
    "poster_country": "JP",
    "poster_year": "1978-01-01"
  }
)
```

This is how to fire a page view event with both above contexts:

```python
t.track_page_view("http://www.films.com", "Homepage", context=[poster_context, geo_context])
```

**Important:** Even if only one custom context is being attached to an event, it still needs to be wrapped in an array.

Note also that you should not pass in an empty array of contexts as this will fail validation. Instead of an empty array, you can pass in `None`.

<a name="tstamp-arg" />

### 4.1.2 Optional timestamp argument

Each `track...()` method supports an optional timestamp as its final argument; this allows you to manually override the timestamp attached to this event. The timestamp should be in milliseconds since the Unix epoch.

If you do not pass this timestamp in as an argument, then the Python Tracker will use the current time to be the timestamp for the event.

Here is an example tracking a structured event and supplying the optional timestamp argument. We can explicitly supply `None`s for the intervening arguments which are empty:

```python
t.track_struct_event("some cat", "save action", None, None, None, 1368725287000)
```

Alternatively, we can use the argument name:

```python
t.track_struct_event("some cat", "save action", tstamp=1368725287000)
```

Timestamp is counted in milliseconds since the Unix epoch - the same format as generated by `time.time() * 1000`.

As of version 0.8.0, providing a `snowplow_tracker.timestamp.TrueTimestamp` object as the timestamp argument will attach a true timestamp to the event, replacing the device timestamp. For example:

```python
from snowplow_tracker.tracker import TrueTimestamp
t.track_struct_event("some cat", "save action", tstamp=TrueTimestamp(1368725287000))
```

Above will attach [`ttm`][python-protocol-datetime]([`true_tstamp`][python-model-datetime]) parameter instead of default `dtm`.
You can also use, plain integer, `DeviceTimestamp` or `None` to send `device_sent_timestamp`.

<a name="return-values" />

### 4.1.3 Tracker method return values

All tracker methods will return the tracker instance, allowing tracker methods to be chained:

```python
e = AsyncEmitter("d3rkrsqld9gmqf.cloudfront.net")
t = Tracker(e)

t.track_page_view("http://www.example.com").track_screen_view("title screen")
```

[Back to top](#top)

<a name="selfdesc-alias" />

### 4.2 Track self-describing events with `track_self_describing_event()`

Use `track_self_describing_event()` to track an event types that you have defined yourself. The arguments are as follows:

| **Argument**   | **Description**                      | **Required?** | **Validation**          |
|---------------:|:-------------------------------------|:--------------|:------------------------|
| `event_json`   | The properties of the event          | Yes           | SelfDescribingJson      |
| `context`      | Custom context for the event         | No            | List(SelfDescribingJson)                    |
| `tstamp`       | When the unstructured event occurred | No            | Positive integer        |

Example:

```python
from snowplow_tracker import SelfDescribingJson

t.track_self_describing_event(SelfDescribingJson(
  "iglu:com.example_company/save-game/jsonschema/1-0-2",
  {
    "save_id": "4321",
    "level": 23,
    "difficultyLevel": "HARD",
    "dl_content": True
  }
))
```

The `event_json` is represented using the SelfDescribingJson class. It has two fields: `schema` and `data`. `data` is a dictionary containing the properties of the unstructured event. `schema` identifies the JSON schema against which `data` should be validated. This schema should be available in your [Iglu schema registry][iglu-schema-registry]  and your Snowplow pipeline configured so that that that registry is included in your Iglu resolver.

For more on JSON schema, see the [blog post][self-describing-jsons].

Many Snowplow users use the above method to track *all* their events i.e. only record event types that they have defined. However, there are a number of "out of the box" events that have dedicated tracking methods. These are detailed below:


<a name="page-view" />

### 4.3 Track pageviews with `track_page_view()`

Use `track_page_view()` to track a user viewing a page within your app or website. The arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `page_url`   | The URL of the page                 | Yes           | Non-empty string        |
| `page_title` | The title of the page               | No            | String                  |
| `referrer`   | The address which linked to the page| No            | String                  |
| `context`    | Custom context for the event        | No            | List(SelfDescribingJson)                    |
| `tstamp`     | When the pageview occurred          | No            | Positive integer        |

Example:

```python
t.track_page_view("www.example.com", "example", "www.referrer.com")
```

[Back to top](#top)

<a name="page-ping" />

### 4.4 Track page pings with `track_page_ping()`

Use `track_page_ping()` to track engagement with a web page over time, via a heartbeat event. (Each ping represents a single heartbeat.)

Arguments are:

| **Argument** | **Description**                                    | **Required?** | **Validation**           |
|-------------:|:---------------------------------------------------|:--------------|:-------------------------|
| `page_url`   | The URL of the page                                | Yes           | Non-empty string         |
| `page_title` | The title of the page                              | No            | String                   |
| `referrer`   | The address which linked to the page               | No            | String                   |
| `min_x`      | Minimum page X offset seen in the last ping period | No            | Positive integer         |
| `max_x`      | Maximum page X offset seen in the last ping period | No            | Positive integer         |
| `min_y`      | Minimum page Y offset seen in the last ping period | No            | Positive integer         |
| `max_y`      | Maximum page Y offset seen in the last ping period | No            | Positive integer         |
| `context`    | Custom context for the event                       | No            | List(SelfDescribingJson) |
| `tstamp`     | When the pageview occurred                         | No            | Positive integer         |


Example:

```py
t.track_page_ping("http://mytesturl/test2", "Page title 2", "http://myreferrer.com", 0, 100, 0, 500, None)
```

<a name="screen-view" />

### 4.5 Track screen views with `track_screen_view()`

Use `track_screen_view()` to track a user viewing a screen (or equivalent) within your app. This is an alternative to the `track_page_view` method which is less web-centric. The arguments are:

| **Argument** | **Description**                     | **Required?** | **Validation**          |
|-------------:|:------------------------------------|:--------------|:------------------------|
| `name`       | Human-readable name for this screen | No           | Non-empty string         |
| `id_`         | Unique identifier for this screen  | No            | String                  |
| `context`    | Custom context for the event        | No            | List(SelfDescribingJson)                    |
| `tstamp`     | When the screen was viewed          | No            | Positive integer        |

Although name and id_ are not individually required, at least one must be provided or the event will fail validation.

Example:

```python
t.track_screen_view("HUD > Save Game", "screen23", null, 1368725287000)
```

[Back to top](#top)



<a name="ecommerce-transaction" />

### 4.6 Track ecommerce transactions with `track_ecommerce_transaction()`

Use `track_ecommerce_transaction()` to track an ecommerce transaction.
Arguments:

| **Argument**     | **Description**                   | **Required?** | **Validation**    |
|-----------------:|:----------------------------------|:--------------|:------------------|
| `order_id`    | ID of the eCommerce transaction      | Yes           | Non-empty string  |
| `total_value` | Total transaction value              | Yes           | Int or Float      |
| `affiliation` | Transaction affiliation              | No            | String            |
| `tax_value`   | Transaction tax value                | No            | Int or Float      |
| `shipping`    | Delivery cost charged                | No            | Int or Float      |
| `city`        | Delivery address city                | No            | String            |
| `state`       | Delivery address state               | No            | String            |
| `country`     | Delivery address country             | No            | String            |
| `currency`    | Transaction currency                 | No            | String            |
| `items`       | Items in the transaction             | Yes           | List              |
| `context`     | Custom context for the event         | No            | List(SelfDescribingJson)              |
| `tstamp`      | When the transaction event occurred  | No            | Positive integer  |

The `items` argument is an array of Python dictionaries representing the items in the transaction. `track_ecommerce_transaction` fires multiple events: one "transaction" event for the transaction as a whole, and one "transaction item" event for each element of the `items` array. Each transaction item event will have the same timestamp, order_id, and currency as the main transaction event.

These are the fields that can appear in a transaction item dictionary:

| **Field**       | **Description**                     | **Required?** | **Validation**           |
|----------------:|:------------------------------------|:--------------|:-------------------------|
| `"sku"`         | Item SKU                            | Yes           | Non-empty string         |
| `"price"`       | Item price                          | Yes           | Int or Float             |
| `"quantity"`    | Item quantity                       | Yes           | Int                      |
| `"name"`        | Item name                           | No            | String                   |
| `"category"`    | Item category                       | No            | String                   |
| `"context"`     | Custom context for the event        | No            | List                     |

Example of tracking a transaction containing two items:

```python
t.track_ecommerce_transaction("6a8078be", 35, city="London", currency="GBP", items=
    [{
        "sku": "pbz0026",
        "price": 20,
        "quantity": 1
    },
    {
        "sku": "pbz0038",
        "price": 15,
        "quantity": 1
    }])
```

[Back to top](#top)

<a name="ecommerce-transaction-item" />

### 4.7 Track ecommerce transactions with `track_ecommerce_transaction_item()`

Use `track_ecommerce_transaction_item()` to track an individual line item within an ecommerce transaction.

Arguments:

| **Argument**  | **Description**                     | **Required?** | **Validation**           |
|--------------:|:------------------------------------|:--------------|:-------------------------|
| `id`          | Order ID                            | Yes           | Non-empty string         |
| `sku`         | Item SKU                            | Yes           | Non-empty string         |
| `price`       | Item price                          | Yes           | Int or Float             |
| `quantity`    | Item quantity                       | Yes           | Int                      |
| `name`        | Item name                           | No            | String                   |
| `category`    | Item category                       | No            | String                   |
| `context`     | Custom context for the event        | No            | List(SelfDescribingJson)                     |
| `tstamp`      | When the transaction event occurred | No            | Positive integer         |

Example:

```python
t.track_ecommerce_transaction_item("order-789", "2001", 49.99, 1, "Green shoes", "clothing")
```

[Back to top](#top)

<a name="struct-event" />

### 4.8 Track structured events with `track_struct_event()`

Use `track_struct_event()` to track a custom event happening in your app which fits the Google Analytics-style structure of having up to five fields (with only the first two required):

| **Argument** | **Description**                                                  | **Required?** | **Validation**           |
|-------------:|:---------------------------------------------------------------  |:--------------|:-------------------------|
| `category`   | The grouping of structured events which this `action` belongs to | Yes           | Non-empty string         |
| `action`     | Defines the type of user interaction which this event involves   | Yes           | Non-empty string         |
| `label`      | A string to provide additional dimensions to the event data      | No            | String                   |
| `property`   | A string describing the object or the action performed on it     | No            | String                   |
| `value`      | A value to provide numerical data about the event                | No            | Int or Float             |
| `context`    | Custom context for the event                                     | No            | List(SelfDescribingJson)                     |
| `tstamp`     | When the structured event occurred                               | No            | Positive integer         |

Example:

```python
t.track_struct_event("shop", "add-to-basket", None, "pcs", 2)
```

[Back to top](#top)



<a name="link-click" />

### 4.9 track_link_click

Use `track_link_click()` to track individual link click events.
Arguments are:

| **Argument**      | **Description**                     | **Required?** | **Validation**          |
|------------------:|:------------------------------------|:--------------|:------------------------|
| `target_url`      | The URL of the page                 | Yes           | Non-empty string        |
| `element_id`      | ID attribute of the HTML element    | No            | String                  |
| `element_classes` | Classes of the HTML element         | No            | List(string)            |
| `element_target`  | Target element                      | No            | String
| `element_content` | The content of the HTML element     | No            | String                  |
| `context`         | Custom context for the event        | No            | List(SelfDescribingJson)|
| `tstamp`          | When the pageview occurred          | No            | Positive integer        |

Basic example:

```python
t.track_link_click("http://my-target-url2/path")
```

Advanced example:

```python
t.track_link_click("http://my-target-url2/path", "element id 2", None, "element target", "element content")
```

<a name="add-to-cart" />

### 4.10 track_add_to_cart

Use `track_add_to_cart()` to track adding items to a cart on an ecommerce site.
Arguments are:

| **Argument** | **Description**               | **Required?** | **Validation**           |
|-------------:|:------------------------------|:--------------|:-------------------------|
| `sku`        | Item SKU or ID                | Yes           | Non-empty string         |
| `quantity`   | Number of items added to cart | Yes           | Integer                  |
| `name`       | Item's name                   | No            | String                   |
| `category`   | Item's category               | No            | String                   |
| `unit_price` | Item's price                  | No            | Int or Float             |
| `currency`   | Currency                      | No            | String                   |
| `context`    | Custom context for the event  | No            | List(SelfDescribingJson) |
| `tstamp`     | When the pageview occurred    | No            | Positive integer         |

Example:

```python
t.track_add_to_cart("123", 2, "The Devil's Dance", "Books", 23.99, "USD", None )
```

<a name="remove-from-cart" />

### 4.11 track_remove_from_cart

Use `track_remove_from_cart()` to track removing items from a cart on an ecommerce site.
Arguments are:

| **Argument** | **Description**               | **Required?** | **Validation**           |
|-------------:|:------------------------------|:--------------|:-------------------------|
| `sku`        | Item SKU or ID                | Yes           | Non-empty string         |
| `quantity`   | Number of items added to cart | Yes           | Integer                  |
| `name`       | Item's name                   | No            | String                   |
| `category`   | Item's category               | No            | String                   |
| `unit_price` | Item's price                  | No            | Int or Float             |
| `currency`   | Currency                      | No            | String                   |
| `context`    | Custom context for the event  | No            | List(SelfDescribingJson) |
| `tstamp`     | When the pageview occurred    | No            | Positive integer         |

Basic example:

```python
t.track_remove_from_cart("123", 1)
```

Advanced example:

```python
t.track_remove_from_cart("123", 2, "The Devil's Dance", "Books", 23.99, "USD")
```

<a name="form-change" />

### 4.12 track_form_change

Use `track_from_change()` to track changes in website form inputs over session.
Arguments are:

| **Argument**      | **Description**                      | **Required?** | **Validation**                                 |
|------------------:|:-------------------------------------|:--------------|:-----------------------------------------------|
| `form_id`         | ID attribute of the HTML form        | Yes           | Non-empty string                               |
| `element_id`      | ID attribute of the HTML element     | Yes           | String                                         |
| `node_name`       | Type of input element                | Yes           | [Valid node_name][change-form-schema-nodename] |
| `value`           | Value of input element               | Yes           | String                                         |
| `type_`           | Type of data the element represents  | No            | Non-empty string                               |
| `element_classes` | Classes of the HTML element          | No            | List(string)                                   |
| `context`         | Custom context for the event         | No            | List(SelfDescribingJson)                       |
| `tstamp`          | When the pageview occurred           | No            | Positive integer                               |

Basic example:

```python
t.track_form_change("signupForm", "ageInput", "age", "24")
```

Advanced example:

```python
t.track_form_change("signupForm", "ageInput", "age", "24", "number", ["signup__number", "form__red"])
```

<a name="form-submit" />

### 4.13 track_form_submit

Use `track_form_submit()` to track sumbitted forms.
Arguments are:

| **Argument**      | **Description**                      | **Required?** | **Validation**          |
|------------------:|:-------------------------------------|:--------------|:------------------------|
| `form_id`         | ID attribute of the HTML form        | Yes           | Non-empty string        |
| `form_classes`    | Classes of the HTML form             | No            | List(str)               |
| `elements`        | Value of input element               | No            | List(dict)              |
| `context`         | Custom context for the event         | No            | List(SelfDescribingJson)                    |
| `tstamp`          | When the pageview occurred           | No            | Positive integer        |

Basic example:

```python
t.track_form_submit("registrationForm")
```

Advanced example:

```python
t.track_form_submit("signupForm", ["signup__warning"], {"name": "email", "value": "tracker@example.com", "nodeName": "INPUT", "type": "email"})
```

<a name="site-search" />

### 4.14 track_site_search

Use `track_site_search()` to track a what user searches on your website.
Arguments are:

| **Argument**    | **Description**                     | **Required?** | **Validation**           |
|----------------:|:------------------------------------|:--------------|:-------------------------|
| `terms`         | Search terms                        | Yes           | List(str)                |
| `filters`       | Filters applied to search           | No            | List(dict{str:str|bool}) |
| `total_results` | Total number of results             | No            | Integer                  |
| `page_results`  | Number of pages of results          | No            | Integer                  |
| `context`       | Custom context for the event        | No            | List(SelfDescribingJson) |
| `tstamp`        | When the pageview occurred          | No            | Positive integer         |

Basic example:

```python
t.track_site_search(["analytics", "snowplow", "tracker"])
```

Advanced example:

```python
t.track_site_search(["pulp fiction", "reviews"], {"nswf": true}, 215, 22)
```

<a name="unstruct-event" />

### 4.15 Track unstructured events with `track_unstruct_event()`

This functionally is equivalent to `track_self_describing_event`. We believe that the method name is misleading: this method is used to track events that are structured in nature (they have an associated schema), which is why we believe referring to them as `self-describing` events makes more sense than referring to them as `unstructured events`.

The method is provided for reasons of backwards compatibility.


<a name="emitters" />

## 5. Emitters

Tracker instances must be initialized with an emitter. This section will go into more depth about the Emitter class and its subclasses.

<a name="base-emitter" />

## 5.1 The basic Emitter class

At its most basic, the Emitter class only needs a collector URI:

```python
from snowplow_tracker import Emitter

e = Emitter("d3rkrsqld9gmqf.cloudfront.net")
```

This is the signature of the constructor for the base Emitter class:

```python
def __init__(self, endpoint,
             protocol="http", port=None, method="get",
             buffer_size=None, on_success=None, on_failure=None):
```

| **Argument**   | **Description**                                | **Required?** | **Validation**              |
|---------------:|:-----------------------------------------------|:--------------|:----------------------------|
| `endpoint`     | The collector URI                              | Yes           | Dict                        |
| `protocol`     | Request protocol: HTTP or HTTPS                | No            | List                        |
| `port`         | The port to connect to                         | No            | Positive integer            |
| `method`       | The method to use: "get" or "post"             | No            | String                      |
| `buffer_size`  | Number of events to store before flushing      | No            | Positive integer            |
| `on_success`   | Callback executed when a flush is successful   | No            | Function taking 1 argument  |
| `on_failure`   | Callback executed when a flush is unsuccessful | No            | Function taking 2 arguments |
| `byte_limit`   | Number of bytes to store before flushing       | No            | Positive integer            |

`protocol` defaults to "http" but can also be "https".

When the emitter receives an event, it adds it to a buffer. When the queue is full, all events in the queue get sent to the collector. The `buffer_size` argument allows you to customize the queue size. By default, it is 1 for GET requests and 10 for POST requests. (So in the case of GET requests, each event is fired as soon as the emitter receives it.) If the emitter is configured to send POST requests, then instead of sending one for every event in the buffer, it will send a single request containing all those events in JSON format.

`on_success` is an optional callback that will execute whenever the queue is flushed successfully, that is, whenever every request sent has status code 200. It will be passed one argument: the number of events that were sent.

`on_failure` is similar, but executes when the flush is not wholly successful. It will be passed two arguments: the number of events that were successfully sent, and an array of unsent requests. (If the emitter is configured to send POST requests, the array will actually be a string, but it can be turned back into an array of Python dictionaries (each corresponding to an event) by using `json.loads`.)

`byte_limit` is similar to `buffer_size`, but instead of counting events - it takes into account only the amount of bytes to be sent over the network. *Warning*: this limit is approximate with infelicity < 1%.

An example:

```python
def f(x):
    print(str(x) + " events sent successfully!")

unsent_events = []

def g(x, y):
    print(str(x) + " events sent successfully!")
    print("These events were not sent successfully and have been stored in unsent_events:")
    for event_dict in y:
        print(event_dict)
        unsent_events.append(event_dict)

e = Emitter("d3rkrsqld9gmqf.cloudfront.net", buffer_size=3, on_success=f, on_failure=g)

t = Tracker(e)

# This doesn't cause the emitter to send a request because the buffer_size was set to 3, not 1
t.track_page_view("http://www.example.com")
t.track_page_view("http://www.example.com/page1")

# This does cause the emitter to try to send all 3 events
t.track_page_view("http://www.example.com/page2")

# Since the method is GET by default, 3 separate requests are sent
# If any of them are unsuccessful, they will be stored in the unsent_events variable
```

<a name="async-emitter" />

## 5.2 The AsyncEmitter class

```python
from snowplow_tracker import AsyncEmitter

e = Emitter("d3rkrsqld9gmqf.cloudfront.net", thread_count=10)
```

The `AsyncEmitter` class works just like the Emitter class. It has one advantage, though: HTTP(S) requests are sent asynchronously, so the Tracker won't be blocked while the Emitter waits for a response. For this reason, the AsyncEmitter is recommended over the base `Emitter` class.

The AsyncEmitter uses a fixed-size thread pool to perform network I/O. By default, this pool contains only one thread, but you can configure the number of threads in the constructor using the `thread_count` argument.

<a name="celery-emitter" />

## 5.3 The CeleryEmitter class

The `CeleryEmitter` class works just like the base `Emitter` class, but it registers sending requests as a task for a [Celery][celery] worker. If there is a module named snowplow_celery_config.py on your PYTHONPATH, it will be used as the Celery configuration file; otherwise, a default configuration will be used. You can run the worker using this command:

```bash
celery -A snowplow_tracker.emitters worker --loglevel=debug
```

Note that `on_success` and `on_failure` callbacks cannot be supplied to this emitter.

<a name="redis-emitter" />

## 5.4 The RedisEmitter class

Use a RedisEmitter instance to store events in a [Redis][redis] database for later use. This is the RedisEmitter constructor function:

```python
def __init__(self, rdb=None, key="snowplow"):
```

`rdb` should be an instance of either the `Redis` or `StrictRedis` class, found in the `redis` module. If it is not supplied, a default will be used. `key` is the key used to store events in the database. It defaults to "snowplow". The format for event storage is a Redis list of JSON strings.

An example:

```python
from snowplow_tracker import RedisEmitter, Tracker
import redis

rdb = redis.StrictRedis(db=2)

e = RedisEmitter(rdb, "my_snowplow_key")

t = Tracker(e)

t.track_page_view("http://www.example.com")

# Check that the event was stored in Redis:
print(rdb.lrange("my_snowplowkey", 0, -1))
# prints something like:
# ['{"tv":"py-0.4.0", "ev": "pv", "url": "http://www.example.com", "dtm": 1400252420261, "tid": 7515828, "p": "pc"}']
```

<a name="manual-flushing" />

### 5.5 Manual flushing

You can flush the emitter manually using the `flush` method of the `Tracker` instance which is sending events to the emitter. This is a blocking call which synchronously sends all events in the emitter's buffer.

```python
t.flush()
```

You can alternatively perform an asynchronous flush, which tells the tracker to send all buffered events but doesn't wait for the sending to complete:

```python
t.flush(False)
```

If you are using the AsyncEmitter, you shouldn't perform a synchronous flush inside an on_success or on_failure callback function as this can cause a deadlock.

<a name="multiple-emitters" />

### 5.6 Multiple emitters

You can configure a tracker instance to send events to multiple emitters by passing the `Tracker` constructor function an array of emitters instead of a single emitter, or by using the `addEmitter` method:

```python
from snowplow_tracker import Subject, Tracker, AsyncEmitter, RedisEmitter
import redis

e1 = AsyncEmitter("collector1.cloudfront.net", method="get")
e1 = AsyncEmitter("collector2.cloudfront.net", method="post")

t = Tracker([e1, e2])

rdb = redis.StrictRedis(db=2)

e3 = RedisEmitter(rdb, "my_snowplow_key")

t.addEmitter(e3)
```

<a name="custom-emitters" />

### 5.7 Custom emitters

You can create your own custom emitter class, either from scratch or by subclassing one of the existing classes (with the exception of `CeleryEmitter`, since it uses the `pickle` module which doesn't work correctly with a class subclassed from a class located in a different module). The only requirement for compatibility is that is must have an `input` method which accepts a Python dictionary of name-value pairs.

<a name="onfailure-loop" />

### 5.8 Automatically retry sending failed events

You can use the following function as the `on_failure` callback to immediately retry failed events:

```python
def on_failure_retry(failed_event_count, failed_events):
  # possible backoff-and-retry timeout here
  for e in failed_events:
    my_emitter.input(e)
```

You may wish to add backoff logic to delay the resending.

<a name="set-flush-timer">

### 5.9 Setting flush timer

You can flush your emitter based on some time interval:

```python
e1 = AsyncEmitter("collector1.cloudfront.net", method="post")
e1.set_flush_timer(5)  # flush each 5 seconds
```

Automatic flush can also be cancelled:

```
e1.cancel_flush()
```

<a name="contracts" />

## 6. Contracts

Python is a dynamically typed language, but each of our methods expects its arguments to be of specific types and value ranges and validates that to be the case. These checks are done using the [PyContracts][pycontracts] library.

If the validation check fails, then a runtime error is thrown:

```python
s = Subject()
t.set_color_depth("walrus")
```
```
contracts.interface.ContractNotRespected: Breach for argument 'depth' to Subject:set_color_depth().
Expected type 'int', got 'str'.
checking: Int      for value: Instance of str: 'walrus'
checking: $(Int)   for value: Instance of str: 'walrus'
checking: int      for value: Instance of str: 'walrus'
Variables bound in inner context:
- self: Instance of Tracker: <snowplow_tracker.tracker.Tracker object...> [clip]

```

If your value is of the wrong type, convert it before passing it into the `track...()` method, for example:

```python
level_idx = 42
t.track_screen_view("Game Level", str(level_idx))
```

You can turn off type checking to improve performance like this:

```python
from snowplow_tracker import disable_contracts
disable_contracts()
```

[Back to top](#top)

<a name="logging" />

## 7. Logging

The emitters.py module has Python logging turned to give you information about requests being sent. The logger prints messages about what emitters are doing. By default, only messages with priority "INFO" or higher will be logged.

To change this:

```python
from snowplow_tracker import logger

# Log all messages, even DEBUG messages
logger.setLevel(10)

# Log only messages with priority WARN or higher
logger.setLevel(30)

# Turn off all logging
logger.setLevel(60)
```

[Back to top](#top)

<a name="redis-worker" />

## 8. The RedisWorker class

The tracker comes with a RedisWorker class which sends Snowplow events from Redis to an emitter. The RedisWorker constructor is similar to the RedisEmitter constructor:

```python
def __init__(self, _consumer, key=None, dbr=None):
```

This is how it is used:

```python
from snowplow_tracker import AsyncEmitter
from snowplow_tracker.redis_worker import RedisWorker

e = Emitter("d3rkrsqld9gmqf.cloudfront.net")

r = RedisWorker(e, key="snowplow_redis_key")

r.run()
```

This will set up a worker which will run indefinitely, taking events from the Redis list with key "snowplow_redis_key" and inputting them to an AsyncEmitter, which will send them to a Cloudfront collector. If the process receives a SIGINT signal (for example, due to a Ctrl-C keyboard interrupt), cleanup will occur before exiting to ensure no events are lost.

[Back to top](#top)

[python-0.2]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.2
[python-0.3]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.3
[python-0.4]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.4
[python-0.5]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.5
[python-0.6]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.6
[python-0.7]: https://github.com/snowplow/snowplow/wiki/Python-Tracker-v0.7
[pycontracts]: http://andreacensi.github.io/contracts/

[jsonschema]: http://snowplowanalytics.com/blog/2014/05/13/introducing-schemaver-for-semantic-versioning-of-schemas/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[base64]: https://en.wikipedia.org/wiki/Base64
[celery]: http://www.celeryproject.org/
[redis]: http://redis.io/
[django-request]: https://docs.djangoproject.com/en/1.7/ref/request-response/

[python-protocol-datetime]: https://github.com/snowplow/snowplow/wiki/snowplow-tracker-protocol#12-date--time-parameter
[python-model-datetime]: https://github.com/snowplow/snowplow/wiki/canonical-event-model#212-date--time-fields

[iglu-schema-registry]: https://github.com/snowplow/iglu
[iglu-central]: https://github.com/snowplow/iglu-central

[change-form-schema-nodename]: https://github.com/snowplow/iglu-central/blob/53589a5dd41f2f88cabb359488019e0ebb72ec49/schemas/com.snowplowanalytics.snowplow/change_form/jsonschema/1-0-0#L20
