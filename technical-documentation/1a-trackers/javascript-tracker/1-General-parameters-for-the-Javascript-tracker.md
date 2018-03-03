<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Trackers**](trackers) » [**JavaScript Tracker**](Javascript-Tracker) » General parameters

*This page refers to version 2.9.0 of the Snowplow JavaScript Tracker.*

*Click [here][general-parameters-v1] for the corresponding documentation for version 1.*

*Click [here][general-parameters-v2.0] for the corresponding documentation for version 2.1.1.*

*Click [here][general-parameters-v2.2] for the corresponding documentation for version 2.2.2.*

*Click [here][general-parameters-v2.3] for the corresponding documentation for version 2.3.0.*

*Click [here][general-parameters-v2.4] for the corresponding documentation for version 2.4.3.*

*Click [here][general-parameters-v2.5] for the corresponding documentation for version 2.5.3.*

*Click [here][general-parameters-v2.6] for the corresponding documentation for version 2.6.2.*

*Click [here][general-parameters-v2.7] for the corresponding documentation for version 2.7.2.*

*Click [here][general-parameters-v2.8] for the corresponding documentation for version 2.8.2.*

<a name="general"></a>

## 2. General parameters

  - 2.1 [Loading Snowplow.js](#loading)
  - 2.2 [Initialising a tracker](#initialisation)
    - 2.2.1 [Setting the application ID](#app-id)
    - 2.2.2 [Setting the platform](#platform)
    - 2.2.3 [Configuring the cookie domain](#cookie-domain)
    - 2.2.4 [Configuring the cookie name](#cookie-name)
    - 2.2.5 [Configuring base 64 encoding](#base-64)
    - 2.2.6 [Respecting Do Not Track](#respect-do-not-track)
    - 2.2.7 [Opt-out cookie](#opt-out)
    - 2.2.8 [User fingerprinting](#user-fingerprint)
    - 2.2.9 [Setting the user fingerprint seed](#user-fingerprint-seed)
    - 2.2.10 [Setting the page unload pause](#page-unload-timer)
    - 2.2.11 [Setting the event request protocol](#force-secure-tracker)
    - 2.2.12 [Setting an unsecure event request protocol](#force-unsecure-tracker)
    - 2.2.13 [Configuring the session cookie duration](#session-cookie-duration)
    - 2.2.14 [Configuring the storage strategy](#storage-strategy)
    - 2.2.15 [Adding predefined contexts](#predefined-contexts)
      - 2.2.15.1 [webPage context](#webPage)
      - 2.2.15.2 [performanceTiming context](#performanceTiming)
      - 2.2.15.3 [gaCookies context](#gaCookies)
      - 2.2.15.4 [geolocation context](#geolocation)
      - 2.2.15.5 [augurIdentityLite context](#augurIdentityLite)
      - 2.2.15.6 [optimizelyExperiments context](#optimizelyExperiments)
      - 2.2.15.7 [optimizelyStates context](#optimizelyStates)
      - 2.2.15.8 [optimizelyVariations context](#optimizelyVariations)
      - 2.2.15.9 [optimizelyVisitor context](#optimizelyVisitor)
      - 2.2.15.10 [optimizelyAudiences context](#optimizelyAudiences)
      - 2.2.15.11 [optimizelyDimensions context](#optimizelyDimensions)
      - 2.2.15.12 [optimizelySummary context](#optimizelySummary)
      - 2.2.15.13 [optimizelyXSummary context](#optimizelyXSummary)
      - 2.2.15.14 [parrable context](#parrable)
    - 2.2.16 [POST support](#post)
    - 2.2.17 [Configuring cross-domain tracking](#cross-domain)
    - 2.2.18 [Configuring the maximum payload size in bytes](#maxPostBytes)
    - 2.2.19 [Automatically discover and set the root domain](#discoverRootDomain)
    - 2.2.20 [Configuring the cookie lifetime](#visitorCookieDuration)
    - 2.2.21 [Tracking prerendered pages](#prerenderedTracking)
  - 2.3 [Other parameters](#other-methods)
    - 2.3.1 [Setting the user id](#user-id)
      - 2.3.1.1 [`setUserId`](#set-user-id)
      - 2.3.1.2 [`setUserIdFromLocation`](#set-user-id-from-location)
      - 2.3.1.3 [`setUserIdFromReferrer`](#set-user-id-from-referrer)
      - 2.3.1.4 [`setUserIdFromCookie`](#set-user-id-from-cookie)
    - 2.3.2 [Setting a custom page URL and referrer URL](#custom-url)
  - 2.4 [Setting onload callbacks](#callback)
  - 2.5 [Managing multiple trackers](#multiple-trackers)
  - 2.6 [How the Tracker stores state](#state)
  - 2.7 [Getting the user ID from the first-party cookie](#get-id)
  - 2.8 [Optional timestamp argument](#timestamp)
  - 2.9 [Preserving pageViewId](#preservePageViewId)

<a name="loading" />

### 2.1 Loading Snowplow.js

Use the following tag to your page to load Snowplow.js:

```html
<script type="text/javascript" async=1>
;(function(p,l,o,w,i,n,g){if(!p[i]){p.GlobalSnowplowNamespace=p.GlobalSnowplowNamespace||[];
p.GlobalSnowplowNamespace.push(i);p[i]=function(){(p[i].q=p[i].q||[]).push(arguments)
};p[i].q=p[i].q||[];n=l.createElement(o);g=l.getElementsByTagName(o)[0];n.async=1;
n.src=w;g.parentNode.insertBefore(n,g)}}(window,document,"script","//d1fc8wv8zag5ca.cloudfront.net/2.9.0/sp.js","snowplow_name_here"));
</script>
```

*Important note regarding testing:* `"//d1fc8wv8zag5ca.cloudfront.net/2.9.0/sp.js"` is the protocol-relative URL used to fetch `sp.js`. It will work if the your web page is using the "http" or "https" protocol. But if you are testing locally and loading your page from your filesystem using the "file" protocol (so its URI looks something like "file:///home/joe/snowplow_test.html"), the protocol-relative URL will also use that protocol, preventing the script from loading. To avoid this, change the URL to `"http://d1fc8wv8zag5ca.cloudfront.net/2.9.0/sp.js"` when testing locally.

As well as loading Snowplow, this tag creates a global function called "snowplow_name_here" which you use to access the Tracker. You can replace the string "snowplow_name_here" with the function name of your choice. This is encouraged: if there are two Snowplow users on the same page, there won't be any conflict between them as long as they have chosen different function names. The rest of the documentation will assume that the function is called "snowplow_name_here".

Once the function `snowplow_name_here` is created, the syntax for using Snowplow methods is as follows:

```javascript
snowplow_name_here({{"methodName"}}, {{first method argument}}, {{second method argument}}, ...);
```

For example, the method `trackStructEvent` has this signature:

```javascript
function trackStructEvent(category, action, label, property, value, context)
```

where only the first two arguments are required. You would use it like this:

```javascript
snowplow_name_here('trackStructEvent', 'Mixes', 'Play', '', '', 20);
```

Empty strings are provided for the `label` and `property` arguments to pad them out. (`Null` would also work.) They won't be added to the event. Neither will the context argument, which isn't provided at all.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="initialisation" />

### 2.2 Initialising a tracker

Tracker initialization is indicated with the `"newTracker"` string and takes three arguments:

1. The tracker namespace
2. The collector endpoint
3. An optional argmap containing other settings

Here is a simple example of how to initialise a tracker:

```javascript
snowplow_name_here("newTracker", "cf", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
});
```

The tracker will be named "cf" and will send events to d3rkrsqld9gmqf.cloudfront.net, the cloudfront collector specified. The final argument is called the argmap. Here it is just used to set the app ID and platform for the tracker. Each event the tracker sends will have an app ID field set to "cfe23a" and a platform field set to "mob".

Here is a longer example in which every tracker configuration parameter is set:

```javascript
snowplow_name_here("newTracker", "cf", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
  cookieDomain: null,
  discoverRootDomain: true,
  cookieName: "_sp534_",
  encodeBase64: false,
  respectDoNotTrack: false,
  userFingerprint: true,
  userFingerprintSeed: 6385926734,
  pageUnloadTimer: 0,
  forceSecureTracker: true,
  post: true,
  bufferSize: 5,
  maxPostBytes: 45000,
  crossDomainLinker: function (linkElement) {
    return (linkElement.href === "http://acme.de" || linkElement.id === "crossDomainLink");
  },
  cookieLifetime: 86400 * 31,
  stateStorageStrategy: "cookie",
  contexts: {
    webPage: true,
    performanceTiming: true,
    gaCookies: true,
    geolocation: false
  }
});
```

We will now go through the various argmap parameters. Note that these are all optional. In fact, you aren't required to provide any argmap at all.

<a name="app-id" />

#### 2.2.1 Setting the application ID

Set the application ID using the `appId` field of the argmap. This will be attached to every event the tracker fires. You can set different application IDs on different parts of your site. You can then distinguish events that occur on different applications by grouping results based on `application_id`.

<a name="platform" />

#### 2.2.2 Setting the platform
Set the application platform using the `platform` field of the argmap. This will be attached to every event the tracker fires. Its default value is "web". For a list of supported platforms, please see the [Snowplow Tracker Protocol][snowplow-tracker-protocol].

<a name="cookie-domain" />

#### 2.2.3 Configuring the cookie domain

If your website spans multiple subdomains e.g.

* www.mysite.com
* blog.mysite.com
* application.mysite.com

You will want to track user behaviour across all those subdomains, rather than within each individually. As a result, it is important that the domain for your first party cookies is set to '.mysite.com' rather than 'www.mysite.com'. By doing so, any values that are stored on the cookie on one of subdomain will be accessible on all the others.

It is recommended that you [enable automatic discovery and setting of the root domain](#discoverRootDomain). 

Otherwise, set the cookie domain for the tracker instance using the `cookieDomain` field of the argmap. If this field is not set, the cookies will not be given a domain.

**WARNING**: *Changing the cookie domain will reset all existing cookies. As a result, it might be a major one-time disruption to data analytics because all visitors to the website will receive a new `domain_userid`.*

<a name="cookie-name" />

#### 2.2.4 Configuring the cookie name

Set the cookie name for the tracker instance using the `cookieName` field of the argmap. The default is "_sp_". Snowplow uses two cookies, a domain cookie and a session cookie. In the default case, their names are "_sp_id" and "_sp_ses" respectively. If you are upgrading from an earlier version of Snowplow, you should use the default cookie name so that the cookies set by the earlier version are still remembered. Otherwise you should provide a new name to prevent clashes with other Snowplow users on the same page.

Once set, you can retrieve a cookie name thanks to the `getCookieName(basename)`
method where basename is id or ses for the domain and session cookie
respectively. As an example, you can retrieve the complete name of the domain
cookie with `getCookieName('id')`.

<a name="base-64">

#### 2.2.5 Configuring base 64 encoding
By default, self-describing events and custom contexts are encoded into Base64 to ensure that no data is lost or corrupted. You can turn encoding on or off using the `encodeBase64` field of the argmap.

<a name="respect-do-not-track" />

#### 2.2.6 Respecting Do Not Track

Most browsers have a Do Not Track option which allows users to express a preference not to be tracked. You can respect that preference by setting the `respectDoNotTrack` field of the argmap to `true`. This prevents cookies from being sent and events from being fired.

<a name="opt-out" />

#### 2.2.7 Opt-out cookie

It is possible to set an opt-out cookie in order not to track anything similarly
to Do Not Track through `setOptOutCookie(cookieName)`. If this cookie is set,
cookies won't be stored and events won't be fired.

<a name="user-fingerprint" />

#### 2.2.8 User fingerprinting

By default, the tracker generates a user fingerprint based on various browser features. This fingerprint is likely to be unique and so can be used to track anonymous users. You can turn user fingerprinting off by setting the `userFingerprint` field of the argmap to `false`.

<a name="user-fingerprint-seed" />

#### 2.2.9 Setting the user fingerprint seed

The `userFingerprintSeed` field of the the argmap lets you choose the hash seed used to generate the user fingerprint. If this is not specified, the default is 123412414.

<a name="page-unload-timer" />

#### 2.2.10 Setting the page unload pause

Whenever the Snowplow Javascript Tracker fires an event, it automatically starts a 500 millisecond timer running. If the user clicks on a link or refreshes the page during this period (or, more likely, if the event was triggered by the user clicking a link), the page will wait until either the event is sent or the timer is finished before unloading. 500 milliseconds is usually enough to ensure the event has time to be sent.

You can change the pause length (in milliseconds) using the `pageUnloadTimer` of the argmap. The above example completely eliminates the pause. This does make it unlikely that events triggered by link clicks will be sent.

See also [How the Tracker uses `localStorage`](#local-storage) for an explanation of how the tracker can later recover and send unsent events.

<a name="force-secure-tracker" />

#### 2.2.11 Setting the event request protocol

Normally the protocol (http or https) used by the Tracker to send events to a collector is the same as the protocol of the current page. You can force it to use https by setting the `forceSecureTracker` field of the argmap to `true`.

<a name="force-unsecure-tracker" />

#### 2.2.12 Setting an unsecure event request protocol

Normally the protocol (http or https) used by the Tracker to send events to a collector is the same as the protocol of the current page. You can force it to use http by setting the `forceUnsecureTracker` field of the argmap to `true`.  If `forceSecureTracker` is activated this argument is ignored.

__NOTE__: This argument should only be used for testing purposes as it creates security vulnerabilities.

<a name="session-cookie-duration" />

#### 2.2.13 Configuring the session cookie duration

Whenever an event fires, the Tracker creates a session cookie. If the cookie didn't previously exist, the Tracker interprets this as the start of a new session.

By default the session cookie expires after 30 minutes. This means that a user leaving the site and returning in under 30 minutes does not change the session. You can override this default by setting `sessionCookieTimeout` to a duration (in seconds) in the argmap. For example,

```javascript
{
  ...
  sessionCookieTimeout: 3600
  ...
}
```

would set the session cookie lifespan to an hour.

<a name="storage-strategy" />

#### 2.2.14 Configuring the storage strategy

Three strategies are made available to store the Tracker's state: cookies, local
storage or no storage at all. You can set the strategy with the help of the
`stateStorageStrategy` parameter in the argmap to "cookie" (the default),
"localStorage" or "none" respectively.

When choosing local storage, the Tracker will additionally store events in
local storage before sending them so that they can be recovered if the user
leaves the page before they are sent.

<a name="predefined-contexts" />

#### 2.2.15 Adding predefined contexts

The JavaScript Tracker comes with many predefined contexts which you can automatically add to every event you send. To enable them, simply add them to the `contexts` field of the argmap as above.

<a name="webPage" />

##### 2.2.15.1 webPage context

When the JavaScript Tracker loads on a page, it generates a new page view UUID. If the webPage context is enabled, then a context containing this UUID is attached to every page view.

<a name="performanceTiming" />

##### 2.2.15.2 performanceTiming context

If this context is enabled, the JavaScript Tracker will use the create a context JSON from the `window.performance.timing` object, along with the Chrome `firstPaintTime` field (renamed to `"chromeFirstPaint"`) if it exists. This data can be used to calculate page performance metrics.

Note that if you fire a page view event as soon as the page loads, the `domComplete`, `loadEventStart`, `loadEventEnd`, and `chromeFirstPaint` metrics in the Navigation Timing API may be set to zero. This is because those properties are only known once all scripts on the page have finished executing. See the [Advanced Usage](3-Advanced-usage-of-the-JavaScript-tracker#timing) page for more information on circumventing this limitation. Additionally the `redirectStart`, `redirectEnd`, and `secureConnectionStart` are set to 0 if there is no redirect or a secure connection is not requested.

For more information on the Navigation Timing API, see [the specification][performance-spec].

<a name="gaCookies" />

##### 2.2.15.3 gaCookies context

If this context is enabled, the JavaScript Tracker will look for Google Analytics cookies (specifically the "__utma", "__utmb", "__utmc", "__utmv", "__utmz", and "_ga" cookies) and combine their values into a JSON which gets sent with every event.

<a name="geolocation" />

##### 2.2.15.4 geolocation context

If this context is enabled, the JavaScript Tracker will attempt to create a context from the visitor's geolocation information. If the visitor has not already given or denied the website permission to use their geolocation information, a prompt will appear. If they give permission, then all events from that moment on will include their geolocation information.

For more information on the geolocation API, see [the specification][geolocation-spec].

<a name="augurIdentityLite" />

##### 2.2.15.5 augurIdentityLite context

If this context is enabled the JavaScript Tracker will use the `window['augur']` object to create a context JSON.

To see what will be captured please see the JsonSchema file [io.augur.snowplow/identity_lite/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/io.augur.snowplow/identity_lite/jsonschema/1-0-0).

<a name="optimizelyExperiments" />

##### 2.2.15.6 optimizelyExperiments context

If this context is enabled the JavaScript Tracker will use the `window['optimizely'].data.experiments` object to create an array of context JSONs; one for each sub-object.

To see what will be captured please see the JsonSchema file [com.optimizely/experiment/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.optimizely/experiment/jsonschema/1-0-0).

<a name="optimizelyStates" />

##### 2.2.15.7 optimizelyStates context

If this context is enabled the JavaScript Tracker will use the `window['optimizely'].data.state` object to create an array of context JSONs; one for each sub-object.

To see what will be captured please see the JsonSchema file [com.optimizely/state/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.optimizely/state/jsonschema/1-0-0).

<a name="optimizelyVariations" />

##### 2.2.15.8 optimizelyVariations context

If this context is enabled the JavaScript Tracker will use the `window['optimizely'].data.variations` object to create an array of context JSONs; one for each sub-object.

To see what will be captured please see the JsonSchema file [com.optimizely/variation/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.optimizely/variation/jsonschema/1-0-0).

<a name="optimizelyVisitor" />

##### 2.2.15.9 optimizelyVisitor context

If this context is enabled the JavaScript Tracker will use the `window['optimizely'].data.visitor` object to create a context JSON.

To see what will be captured please see the JsonSchema file [com.optimizely/visitor/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.optimizely/visitor/jsonschema/1-0-0).

<a name="optimizelyAudiences" />

##### 2.2.15.10 optimizelyAudiences context

If this context is enabled the JavaScript Tracker will use the `window['optimizely'].data.visitor.audiences` object to create an array of context JSONs; one for each sub-object.

To see what will be captured please see the JsonSchema file [com.optimizely/visitor_audience/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.optimizely/visitor_audience/jsonschema/1-0-0).

<a name="optimizelyDimensions" />

##### 2.2.15.11 optimizelyDimensions context

If this context is enabled the JavaScript Tracker will use the `window['optimizely'].data.visitor.dimensions` object to create an array of context JSONs; one for each sub-object.

To see what will be captured please see the JsonSchema file [com.optimizely/visitor_dimension/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.optimizely/visitor_dimension/jsonschema/1-0-0).

<a name="optimizelySummary" />

##### 2.2.15.12 optimizelySummary context

Unlike previously mentioned Optimizely contexts this context doesn't attach existing in browser object, but constructs its own using only data necessary to join with [exported Optimizely][optimizely-export] data.

To see what will be captured please see the JsonSchema file [com.optimizely.snowplow/optimizely_summary/jsonschema/1-0-0](https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.optimizely.snowplow/optimizely_summary/jsonschema/1-0-0).

We highly recommend to use this context instead of previous ones because it is has much smaller footprint and contains all necessary data.

<a name="optimizelyXSummary" />

##### 2.2.15.13 optimizelyXSummary context

Support for OptimizelyX has been introduced in the tracker, you can have a look
at the JsonSchema in [com.optimizely/optimizelyx_summary/jsonschema/1-0-0](
https://raw.githubusercontent.com/snowplow/iglu-central/blob/master/schemas/com.optimizely/optimizelyx_summary/jsonschema/1-0-0)
to see what is being captured.

If you're planning on leveraging the context's variation names, you'll have
to untick 'Mask descriptive names in project code and third-party integrations'
in the OptimizelyX menu -> Settings -> Privacy. Otherwise, all variation names
will be `null`.

<a name="parrable" />

##### 2.2.15.14 parrable context

If this context is enabled, the JavaScript Tracker will use the
`window['_hawk']` object to create a [Parrable](https://www.parrable.com/)
context JSON.

To see what will captured, please see the JsonSchema file
[com.parrable/encrypted_payload/jsonschema/1-0-0](
https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.parrable/encrypted_payload/jsonschema/1-0-0).

<a name="post" />

#### 2.2.16 POST support

If you set the `post` field of the argmap to `true`, the tracker will send events using POST requests rather than GET requests. In browsers such as Internet Explorer 9 which do not support cross-origin XMLHttpRequests, the tracker will fall back to using GET.

The main advantage of POST requests is that they circumvent Internet Explorer's maximum URL length of 2083 characters by storing the event data in the body of the request rather than the querystring.

The [Clojure Collector][clojure-collector] and [Scala Stream Collector][ssc] accept events sent by POST; the [Cloudfront Collector][cloudfront-collector] does not..

You can also batch events sent by POST by setting a numeric `bufferSize` field in the argmap. This is the number of events to buffer before sending them all in a single POST. If the user navigates away from the page while the buffer is only partially full, the tracker will attempt to send all stored events immediately, but this often doesn't happen before the page unloads. Normally the tracker will store unsent events in `localStorage`, meaning that unsent events will be resent when the user next visits a page on the same domain. The `bufferSize` defaults to 1, meaning events are sent as soon as they are created.

If you have set `bufferSize` to greater than 1, you can flush the buffer using the `flushBuffer` method:

```
snowplow("flushBuffer");
```

For instance, if you wish to send several events at once, you might make the API calls to create the events and store them and then and call `flushBuffer` afterwards to ensure they are all sent before the user leaves the page.

Note that if `localStorage` is inaccessible or you are not using it to store data, the buffer size will always be 1 to prevent losing events when the user leaves the page.

#### 2.2.17 Configuring cross-domain tracking

The JavaScript Tracker can add an additional parameter named "_sp" to the querystring of outbound links. Its value includes the domain user ID for the current page and the time at which the link was clicked. This makes these values visible in the "url" field of events sent by an instance of the JavaScript Tracker on the destination page. The enrichment process will use these values to populate the `refr_domain_userid` and `refr_dvce_tstamp` fields in Redshift for all events fired on the destination page.

You can configure which links get decorated this way using the `crossDomainLinker` field of the argmap. This field should be a function taking one argument (the link element) and return `true` if the link element should be decorated and false otherwise. For example, this function would only decorate those links whose destination is "http://acme.de" or whose HTML id is "crossDomainLink":

```javascript
{
  crossDomainLinker: function (linkElement) {
    return (linkElement.href === "http://acme.de" || linkElement.id === "crossDomainLink");
  }
}
```

If you want to decorate every link to the domain github.com:

```javascript
{
  crossDomainLinker: function (linkElement) {
    return /^https:\/\/github\.com/.test(linkElement.href);
  }
}
```

If you want to decorate every link, regardless of its destination:

```javascript
{
  crossDomainLinker: function (linkElement) {
    return true;
  }
}
```

Note that the above will decorate "links" which are actually just JavaScript actions (with an `href` of `"javascript:void(0)"`). To exclude these links:

```javascript
window.snowplow('crossDomainLinker', function(linkElement) {
  return linkElement.href.indexOf('javascript:') < 0;
});
```

Note that when the tracker loads, it does not immediately decorate links. Instead it adds event listeners to links which decorate them as soon as a user clicks on them or navigates to them using the keyboard. This ensures that the timestamp added to the querystring is fresh.

If further links get added to the page after the tracker has loaded, you can use the tracker's `crossDomainLinker` method to add listeners again. (Listeners won't be added to links which already have them.)

```javascript
snowplow_name_here('crossDomainLinker', function () {
  return (linkElement.href === "http://acme.de" || linkElement.id === "crossDomainLink");
});
```

*Warning*: If you enable link decoration, you should also make sure that at least one event is fired on the page. Firing an event causes the tracker to write the domain_userid to a cookie. If the cookie doesn't exist when the user leaves the page, the tracker will generate a new ID for them when they return rather than keeping the old ID.

<a name="maxPostBytes" />

#### 2.2.18 Configuring the maximum payload size in bytes

Because the Clojure Collector and the Scala Stream Collector both have a maximum request size, the Tracker limits POST requests to 40000 bytes. If the combined size of the events in `localStorage` is greater than this limit, they will be split into multiple POST requests. You can override this default using a `maxPostBytes` in the argmap.

The Clojure Collector can't handle requests bigger than 64kB. The Scala Stream Collector cannot process requests bigger than 50kB because that is the maximum size of a Kinesis record.

<a name="discoverRootDomain" />

#### 2.2.19 Automatically discover and set the root domain

If the optional `discoverRootDomain` field of the argmap is set to `true`, the Tracker automatically discovers and sets the `configCookieDomain` value to the root domain.

__NOTE__: If you have been setting this manually please note that the automatic detection does not prepend a '.' to the domain. For example a root domain of ".mydomain.com" would become "mydomain.com".  This is because the library we use for setting cookies doesn't care about the difference.

**This will then result in a different domain hash, so we recommend that if you have been setting this manually with a leading '.' to continue to do so manually.**

<a name="visitorCookieDuration" />

#### 2.2.20 Configuring the cookies lifetime

Whenever tracker initialized on your domain - it will set domain-specific visitor's cookies.
By default, these cookies will be active for 2 years.
You can change this duration using `cookieLifetime` argmap parameter or `setVisitorCookieTimeout` method.

```javascript
snowplow_name_here("newTracker", "cf", "d3rkrsqld9gmqf.cloudfront.net", {
  cookieLifetime: 86400 * 31,
});

// or

snowplow_name_here('setVisitorCookieTimeout', 86400 * 30);  // 30 days
```

If `cookieLifetime` is set to `0`, the cookie will expire at the end of the session (when the browser closes).
If set to `-1`, the first-party cookies will be disabled.

<a name="prerenderedTracking" />

#### 2.2.21 Tracking prerendered pages

Some browsers can "preload" pages while user typing URL in.
These users not always end up in that page, however due page preloading tracker is initialized and loaded.

JS Tracker by default doesn't fire events when page is preloaded, but sets callback on [visibilitychange](https://developer.mozilla.org/en-US/docs/Web/Events/visibilitychange) event, which fires actual event only when page starts to render.

To explicitly enable tracking for prerendered pages you can use `setCountPreRendered` function:

```javascript
snowplow_name_here("setCountPreRendered", true);
```


[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="other-methods" />

### 2.3 Other parameters

<a name="user-id" />

#### 2.3.1 Setting the user ID

The JavaScript Tracker automatically sets a `domain_userid` based on a first party cookie.

There are many situations, however, when you will want to identify a specific user using an ID generated by one of your business systems. To do this, you use one of the methods described in this section: `setUserId`, `setUserIdFromLocation`, `setUserIdFromReferrer`, and `setUserIdFromCookie`.

Typically, companies do this at points in the customer journey when the user identifies him / herself e.g. if he / she logs in.

Note: this will only set the user ID on further events fired while the user is on this page; if you want events on another page to record this user ID too, you must call `setUserId` on the other page as well.

<a name="set-user-id" />

##### 2.3.1.1 `setUserId`

`setUserId` is the simplest of the four methods. It sets the business user ID to a string of your choice:

```javascript
snowplow_name_here('setUserId', 'joe.blogs@email.com');
```

Note: `setUserId` can also be called using the alias `identifyUser`.

<a name="set-user-id-from-location" />

##### 2.3.1.1 `setUserIdFromLocation`

`setUserIdFromLocation` lets you set the user ID based on a querystring field of your choice. For example, if the URL is `http://www.mysite.com/home?id=user345`, then the following code would set the user ID to "user345":

```javascript
snowplow_name_here('setUserIdFromLocation', 'id');
```

<a name="set-user-id-from-referrer" />

##### 2.3.1.1 `setUserIdFromReferrer`

`setUserIdFromReferrer` functions in the same way as `setUserIdFromLocation`, except that it uses the referrer querystring rather than the querystring of the current page.

```javascript
snowplow_name_here('setUserIdFromReferrer', 'id');
```

<a name="set-user-id-from-cookie" />

##### 2.3.1.1 `setUserIdFromCookie`

Use `setUserIdFromCookie` to set the value of a cookie as the user ID. For example, if you have a cookie called "cookieid" whose value is "user123", the following code would set the user ID to "user123":

```javascript
snowplow_name_here('setUserIdFromCookie', 'cookieid');
```
[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="custom-url" />

#### 2.3.2 Setting a custom page URL and referrer URL

The Snowplow JavaScript Tracker automatically tracks the page URL and referrerURL on any event tracked. However, in certain situations, you may want to override the one or both of these URLs with a custom value. (For example, this might be desirable if your CMS spits out particularly ugly URLs that are hard to unpick at analysis time.)

To set a custom page URL, use the `setCustomUrl` method:

```javascript
snowplow_name_here('setCustomUrl', 'http://mysite.com/checkout-page');
```

To set a custom referrer, use the `setReferrerUrl` method:

```javascript
snowplow_name_here('setReferrerUrl', 'http://custom-referrer.com');
```

On a single-page app, the page URL might change without the page being reloaded. Whenever an event is fired, the Tracker checks whether the page URL has changed since the last event. If it has, the page URL is updated and the URL at the time of the last event is used as the referrer. If you use `setCustomUrl`, the page URL will no longer be updated in this way. If you use `setReferrerUrl`, the referrer URL will no longer be updated in this way.

If you want to ensure that the original referrer is preserved even though your page URL can change without the page being reloaded, use `setReferrerUrl` like this before sending any events:

```javascript
snowplow_name_here('setReferrerUrl', document.referrer);
```

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="callback" />

### 2.4 Setting onload callbacks

If you call `snowplow_name_here` with a function as the argument, the function will be executed when sp.js loads:

```javascript
snowplow_name_here(function () {
  console.log("sp.js has loaded");
});
```

Or equivalently:

```javascript
snowplow_name_here(function (x) {
  console.log(x);
}, "sp.js has loaded");
```

The callback function should not be a method:

```javascript
// TypeError: Illegal invocation
snowplow_name_here(console.log, "sp.js has loaded");
```

will not work, because the value of `this` in the `console.log` function will be `window` rather than `console`.

You can get around this problem using `Function.prototoype.bind` as follows:

```javascript
snowplow_name_here(console.log.bind(console), "sp.js has loaded");
```

<a name="multiple-trackers" />

### 2.5 Managing multiple trackers

You have more than one tracker instance running on the same page at once. This may be useful if you want to log events to different collectors. By default, any Snowplow method you call will be executed by every tracker you have created so far:

```javascript
snowplow_name_here("newTracker", "cf1", "d3rkrsqld9gmqf.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
});

snowplow_name_here("newTracker", "cf2", "a5grvrhue7ewvt.cloudfront.net", {
  appId: "cfe23a",
  platform: "mob"
});

// Both trackers will use this custom title
snowplow_name_here('setCustomUrl', 'http://mysite.com/checkout-page');

// Both trackers will fire a structured event
snowplow_name_here('trackStructEvent', 'Mixes', 'Play', 'MrC/fabric-0503-mix', '', '0.0');
```

You can override this behaviour and specify which trackers will execute a Snowplow method. To do this, change the method name by adding a colon followed by a list of tracker names separated by semicolons:

```javascript
// Only the first tracker will fire this structured event
snowplow_name_here('trackStructEvent:cf1', 'Mixes', 'Play', 'MrC/fabric-0503-mix', '', '0.0');

// Only the second tracker will fire this self-describing event
snowplow_name_here('trackSelfDescribingEvent:cf2', 'com.acme_company' 'Viewed Product',
    {
        product_id: 'ASO01043',
        category: 'Dresses',
        brand: 'ACME',
        returning: true,
        price: 49.95,
        sizes: ['xs', 's', 'l', 'xl', 'xxl'],
        available_since$dt: new Date(2013,3,7)
    }
);

// Both trackers will fire a page view event
snowplow_name_here('trackPageView:cf1;cf2');
```

<a name="state" />

### 2.6 How the Tracker stores state

Unless you have enabled `respectDoNotTrack` in the configuration argmap, the
tracker will persist information on the client. The location depends on
the `stateStorageStrategy` field in the argmap. By default, information will
be stored in cookie. Alternatively, you can specify "localStorage" to have the
state stored in local storage. Finally, you can set `stateStorageStrategy` to
none in order not to store anything client side.

The stored state takes the form of two first party cookies: the session cookie
and the ID cookie. By default their names are prefixed with "_sp_", but you can
change this using the "cookieName" field in the argmap. Their names are suffixed
with a hash of the current domain, so the full cookie names might look something
like _sp_ses.4209 and _sp_id.4209.

#### The session cookie

Called _sp_ses.{{DOMAIN HASH}} by default, the only purpose of this cookie is to differentiate between different visits. Whenever an event is fired, the session cookie is set to expire in 30 minutes. (This value can be altered using `setSessionCookieTimeout`.)

If no session cookie is already present when an event fires, the tracker treats this as an indication that long enough has passed since the user last visited that this session should be treated as a new session rather than a continuation of the previous session. The `visitCount` (how many times the user has visited) is increased by one and the `lastVisitTs` (the timestamp for the last session) is updated.

Note: A new session can be started at any time by calling the function `newSession`.

#### The ID cookie

This cookie is called _sp_id.{{DOMAIN HASH}} by default. It is used to persist information about a user's activity on the domain between sessions. It contains the following information:

* An ID for the user based on a hash of various browser attributes
* How many times the user has visited the domain
* The timestamp of the user's first visit
* The timestamp of the current visit
* The timestamp of the last visit
* The ID of the current session

It expires after 2 years.

#### The Clojure Collector cookie

There is a third sort of Snowplow-related cookie: the cookie set by the [Clojure Collector][clojure-collector], independently of the JavaScript Tracker. If you are using another type of collector, this cookie will not be set. The Clojure Collector cookie is called "sp". It is a third-party cookie used to track users over multiple domains. It expires after one year.

<a name="get-id" />

### 2.7 Getting the user ID from the Snowplow cookie

You can use the following function to extract the user ID from the ID cookie:

```javascript
/*
* Function to extract the Snowplow user ID from the first-party cookie set by the Snowplow JavaScript Tracker
*
* @param string cookieName (optional) The value used for "cookieName" in the tracker constructor argmap
* (leave blank if you did not set a custom cookie name)
*
* @return string or bool The ID string if the cookie exists or false if the cookie has not been set yet
*/
function getSnowplowDuid(cookieName) {
  cookieName = cookieName || '_sp_';
  var matcher = new RegExp(cookieName + 'id\\.[a-f0-9]+=([^;]+);?');
  var match = document.cookie.match(matcher);
  if (match && match[1]) {
    return match[1].split('.')[0];
  } else {
    return false;
  }
}
```

If you set a custom `cookieName` field in the argmap, pass that name into the function; otherwise call the function without arguments. Note that if the function is called before the cookie exists (i.e. when the user is visiting the page for the first time and sp.js has not yet loaded) if will return `false`.

<a name="timestamp" />

### 2.9 Optional timestamp argument

Since 2.7.0 each `track...()` method supports an optional timestamp as its final argument; this allows you to manually override the timestamp attached to this event.
The timestamp should be in milliseconds since the Unix epoch.

If you do not pass this timestamp in as an argument, then the JavaScript Tracker will use the current time to be the timestamp for the event.

Here is an example tracking a structured event and supplying the optional timestamp argument.

```javascript
snowplow_name_here("trackSelfDescribingEvent", {"schema": "iglu:com.acme/event/jsonschema/1-0-0", "data": {"type": "user_action"}}, [], 1368725287000)
```

Timestamp is counted in milliseconds since the Unix epoch - the same format as generated by `new Date().getTime()`.

Also you can attach a true timestamp to the event, replacing the device timestamp. For example:

```javascript
snowplow_name_here("trackSelfDescribingEvent", {"schema": "iglu:com.acme/event/jsonschema/1-0-0", "data": {"type": "user_action"}}, [], {type: 'ttm', value: 1368725287000})
```

Above will attach `ttm` ([`true_tstamp`][model-datetime]) parameter instead of default `dtm`.
You can also use, plain number or `{type: 'dtm', value: stamp}` to send `device_sent_timestamp`.

<a name="preservePageViewId" />

### 2.10 Preserving pageViewId

As explained in [webPage section](#webPage), JS tracker regenerates `webPage` context each time `trackPageView` was called.
However, before 2.7.0 this was not always the case - `webPage` context was regenerating only when whole HTML page loaded, initialising the tracker.
It means `pageViewId` would remain same on many single-page applications.
This behaviour was considered a bug, but if for some reasons you need restore old behaviour you can use `preservePageViewId` method:

```javascript
snowplow_name_here("preservePageViewId")
```


[contents]: Javascript-Tracker
[general-parameters-v1]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v1
[general-parameters-v2.0]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.0
[general-parameters-v2.2]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.2
[general-parameters-v2.3]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.3
[general-parameters-v2.4]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.4
[general-parameters-v2.5]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.5
[general-parameters-v2.6]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.6
[general-parameters-v2.7]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.7
[general-parameters-v2.8]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker-v2.8
[snowplow-tracker-protocol]: https://github.com/snowplow/snowplow/wiki/SnowPlow-Tracker-Protocol
[contexts]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v1#custom-contexts
[clojure-collector]: https://github.com/snowplow/snowplow/wiki/Clojure-collector
[ssc]: https://github.com/snowplow/snowplow/wiki/scala-stream-collector
[cloudfront-collector]: https://github.com/snowplow/snowplow/wiki/Setting-up-the-Cloudfront-collector
[performancetiming]: https://github.com/snowplow/iglu-central/blob/master/schemas/org.w3/PerformanceTiming/jsonschema/1-0-0
[performance-spec]: http://www.w3.org/TR/2012/REC-navigation-timing-20121217/#sec-window.performance-attribute
[geolocation-spec]: http://dev.w3.org/geo/api/spec-source.html
[optimizely-export]: https://developers.optimizely.com/classic/events/export/index.html
[model-datetime]: https://github.com/snowplow/snowplow/wiki/canonical-event-model#212-date--time-fields