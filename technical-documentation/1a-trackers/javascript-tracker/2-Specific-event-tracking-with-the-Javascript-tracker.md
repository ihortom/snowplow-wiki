<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Trackers**](trackers) » [**JavaScript Tracker**](Javascript-Tracker) » Specific event tracking

*This page refers to version 2.9.0 of the Snowplow JavaScript Tracker.*

*Click [here][specific-events-v1] for the corresponding documentation for version 1.*

*Click [here][specific-events-v2.0] for the corresponding documentation for version 2.0.*

*Click [here][specific-events-v2.2] for the corresponding documentation for version 2.2.2.*

*Click [here][specific-events-v2.3] for the corresponding documentation for version 2.3.0.*

*Click [here][specific-events-v2.4] for the corresponding documentation for version 2.4.3.*

*Click [here][specific-events-v2.5] for the corresponding documentation for version 2.5.3.*

*Click [here][specific-events-v2.6] for the corresponding documentation for version 2.6.2.*

*Click [here][specific-events-v2.7] for the corresponding documentation for version 2.7.2.*

*Click [here][specific-events-v2.8] for the corresponding documentation for version 2.8.2.*

<a name="tracking-specific-events" />

## 3. Tracking specific events

Snowplow has been built to enable users to track a wide range of events that occur when consumers interact with their websites and webapps. We are constantly growing the range of functions available in order to capture that data more richly.

  - 3.1 [Pageviews](#page)
    - 3.1.1 [`trackPageView`](#trackPageView)
  - 3.2 [Pagepings](#pagepings)
    - 3.2.1 [`enableActivityTracking`](#enableActivityTracking)
    - 3.2.2 [`updatePageActivity`](#updatePageActivity)
  - 3.3 [Ecommerce transaction tracking](#ecommerce)
    - 3.3.1 [`addTrans`](#addTrans)
    - 3.3.2 [`addItem`](#addItem)
    - 3.3.3 [`trackTrans`](#trackTrans)
    - 3.3.4 [Pulling it all together: an example](#ecomm-example)
  - 3.4 [Social tracking](#social)
    - 3.4.1 [`trackSocialInteraction`](#trackSocial)
  - 3.5 [Campaign tracking](#campaign)
    - 3.5.1 [Identifying paid sources](#identifying-paid-sources)
    - 3.5.2 [Anatomy of the query parameter](#anatomy-of-the-query-parameters)
  - 3.6 [Ad tracking methods](#ad-tracking)
    - 3.6.1 [`trackAdImpression`](#adImpression)
    - 3.6.2 [`trackAdClick`](#adClick)
    - 3.6.3 [`trackAdConversion`](#adConversion)
    - 3.6.4 [Example: implementing impression tracking with Snowplow and OpenX](#ad-example)
  - 3.7 [Tracking custom structured events](#custom-structured-events)
    - 3.7.1 [`trackStructEvent`](#trackStructEvent)
  - 3.8 [Tracking custom self-describing (unstructured) events](#38-tracking-custom-self-describing-unstructured-events)
    - 3.8.1 [`trackSelfDescribingEvent`](#381-trackselfdescribingevent)
  - 3.9 [Link click tracking](#link-click-tracking)
    - 3.9.1 [`enableLinkClickTracking`](#enableLinkClickTracking)
    - 3.9.2 [`refreshLinkClickTracking`](#refreshLinkClickTracking)
    - 3.9.3 [`trackLinkClick`](#trackLinkClick)
  - 3.10 [Form tracking](#form-tracking)
    - 3.10.1 [`enableFormTracking`](#enableFormTracking)
  - 3.11 [`trackAddToCart` and `trackRemoveFromCart`](#cart)
  - 3.12 [`trackSiteSearch`](#siteSearch)
  - 3.13 [`trackTiming`](#timing)
  - 3.14 [Enhanced Ecommerce tracking](#enhanced-ecommerce)
    - 3.14.1 [`addEnhancedEcommerceActionContext`](#addEnhancedEcommerceActionContext)
    - 3.14.2 [`addEnhancedEcommerceImpressionContext`](#addEnhancedEcommerceImpressionContext)
    - 3.14.3 [`addEnhancedEcommerceProductContext`](#addEnhancedEcommerceProductContext)
    - 3.14.4 [`addEnhancedEcommercePromoContext`](#addEnhancedEcommercePromoContext)
    - 3.14.5 [`trackEnhancedEcommerceAction`](#trackEnhancedEcommerceAction)
  - 3.15 [Consent event tracking](#consent)
    - 3.15.1 [`trackConsentGranted`](#trackConsentGranted)
    - 3.15.2 [`trackConsentWithdrawn`](#trackConsentWithdrawn)
    - 3.15.3 [Consent documents](#consent-documents)
  - 3.16 [Custom contexts](#custom-contexts)
  - 3.17 [Error tracking](#316-error-tracking)
    - 3.17.1 [`trackError`](#trackError)
    - 3.17.2 [`enableErrorTracking`](#enableErrorTracking)
  - 3.18 [Setting the true timestamp](#trueTimestamps)

<a name="page" />

### 3.1 Pageviews

Page views are tracked using the `trackPageView` method. This is generally part of the first Snowplow tag to fire on a particular web page. As a result, the `trackPageView` method is usually deployed straight after the tag that also invokes the Snowplow JavaScript (sp.js) e.g.

```html
<!-- Snowplow starts plowing -->
<script type="text/javascript">
;(function(p,l,o,w,i,n,g){if(!p[i]){p.GlobalSnowplowNamespace=p.GlobalSnowplowNamespace||[];
p.GlobalSnowplowNamespace.push(i);p[i]=function(){(p[i].q=p[i].q||[]).push(arguments)
};p[i].q=p[i].q||[];n=l.createElement(o);g=l.getElementsByTagName(o)[0];n.async=1;
n.src=w;g.parentNode.insertBefore(n,g)}}(window,document,"script","//d1fc8wv8zag5ca.cloudfront.net/2.5.1/sp.js","snowplow_name_here"));

snowplow_name_here('enableActivityTracking', 30, 10);

snowplow_name_here('trackPageView');

 </script>
<!-- Snowplow stops plowing -->
```

<a name="trackPageView" />

#### 3.1.1 `trackPageView`

Track pageview is called using the simple:

```javascript
snowplow_name_here('trackPageView');
```

This method automatically captures the URL, referrer and page title (inferred from the `<title>` tag.

If you wish, you can override the title with a custom value:

```javascript
snowplow_name_here('trackPageView', 'my custom page title');
```

`trackPageView` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

Additionally, you can pass a function which returns an array of zero or more contexts to trackPageView. For the page view and for all subsequent page pings, the function will be called and the contexts it returns will be added to the event.

For example:

```javascript
// Turn on page pings every 10 seconds
window.snowplow('enableActivityTracking', 10, 10);

window.snowplow(
  'trackPageView',

  // no custom title
  null,

  // The usual array of static contexts
  [{
    schema: 'iglu:com.acme/static_context/jsonschema/1-0-0',
    data: {
      staticValue: new Date().toString()
    }
  }],

  // Function which returns an array of custom contexts
  // Gets called once per page view / page ping
  function() {
    return [{
      schema: 'iglu:com.acme/dynamic_context/jsonschema/1-0-0',
      data: {
        dynamicValue: new Date().toString()
      }
    }];
  }
);
```

The page view and every subsequent page ping will have both a static_context and a dynamic_context attached. The static_contexts will all have the same staticValue, but the dynamic_contexts will have different dynamicValues since a new context is created for every event.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="pagepings" />

### 3.2 Track engagement with a web page over time: page pings

As well as tracking page views, we can monitor whether a user continues to engage with a page over time, and record how he / she digests content on the page over time.

That is accomplished using 'page ping' events. If activity tracking is enabled, the web page is monitored to see if a user is engaging with it. (E.g. is the tab in focus, does the mouse move over the page, does the user scroll etc.) If any of these things occur in a set period of time, a page ping event fires, and records the maximum scroll left / right and up / down in the last ping period. If there is no activity in the page (e.g. because the user is on a different tab in his / her browser), no page ping fires.

<a name="enableActivityTracking" />

#### 3.2.1 `enableActivityTracking`

Page pings are enabled by:

```javascript
snowplow_name_here('enableActivityTracking', minimumVisitLength, heartBeat);
```

where `minimumVisitLength` is the time period from page load before the first page ping occurs, in seconds. Heartbeat is the number of seconds between each page ping, once they have started. So, if you executed:

```javascript
snowplow_name_here('enableActivityTracking', 30, 10);

snowplow_name_here('trackPageView');
```

The first ping would occur after 30 seconds, and subsequent pings every 10 seconds as long as the user continued to browse the page actively.

Notes:

* In general this is executed as part of the main Snowplow tracking tag. As a result, you can elect to enable this on specific pages.
* The `enableActivityTracking` method **must** be called *before* the `trackPageView` method.
* Activity tracking will be disabled if either `minimumVisitLength` or
`heartBeat` is not integer. This is to prevent relentless callbacks.

<a name="updatePageActivity" />

#### 3.2.2 `updatePageActivity`

You can also trigger a page ping manually with:

```javascript
snowplow_name_here('updatePageActivity');
```

This is particularly useful when a user is passively engaging with your content,
e.g. watching a video.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="ecommerce" />

### 3.3 Ecommerce tracking

Modelled on Google Analytics ecommerce tracking capability, Snowplow uses three methods that have to be used together to track online transactions:

1. **Create a transaction object**. Use `addTrans()` method to initialize a transaction object. This will be the object that is loaded with all the data relevant to the specific transaction that is being tracked including all the items in the order, the prices of the items, the price of shipping and the `order_id`.
2. **Add items to the transaction.** Use the `addItem()` method to add data about each individual item to the transaction object.
3. **Submit the transaction to Snowplow** using the trackTrans() method, once all the relevant data has been loaded into the object.

<a name="addTrans" >

#### 3.3.1 `addTrans`

The `addTrans` method creates a transaction object. It takes nine possible parameters, two of which are required:

| **Parameter** | **Description**                                      | **Required?** | **Example value** |
|:--------------|:-----------------------------------------------------|:--------------|:------------------|
| `orderId`     | Internal unique order id number for this transaction | Yes           | '1234'            |
| `affiliation` | Partner or store affiliation                         | No            | 'Womens Apparel'  |
| `total`       | Total amount of the transaction                      | Yes           | '19.99'           |
| `tax`         | Tax amount of the transaction                        | No            | '1.00'            |
| `shipping`    | Shipping charge for the transaction                  | No            | '2.99'            |
| `city`        | City to associate with transaction                   | No            | 'San Jose'        |
| `state`       | State or province to associate with transaction      | No            | 'California'      |
| `country`     | Country to associate with transaction                | No            | 'USA'             |
| `currency`    | Currency to associate with this transaction          | No            | 'USD'             |

For example:

```javascript
snowplow_name_here('addTrans',
    '1234',           // order ID - required
    'Acme Clothing',  // affiliation or store name
    '11.99',          // total - required
    '1.29',           // tax
    '5',              // shipping
    'San Jose',       // city
    'California',     // state or province
    'USA',            // country
    'USD'             // currency
  );
```

`addTrans` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="addItem" />

#### 3.3.2 `addItem`

The `addItem` method is used to capture the details of each product item included in the transaction. It should therefore be called once for each item.

There are six potential parameters that can be passed with each call, four of which are required:

| **Parameter** | **Description**                                    | **Required?**                                       | **Example value** |
|:--------------|:---------------------------------------------------|:----------------------------------------------------|:------------------|
| `orderId`     | Order ID of the transaction to associate with item | Yes                                                 | '1234'            |
| `sku`         | Item's SKU code                                    | Yes                                                 | 'pbz0001234'      |
| `name`        | Product name                                       | No, but advisable (to make interpreting SKU easier) | 'Black Tarot'     |
| `category`    | Product category                                   | No                                                  | 'Large'           |
| `price`       | Product price                                      | Yes                                                 | '9.99'            |
| `quantity`    | Purchase quantity                                  | Yes                                                 | '1'               |
| `currency`    | Product price currency                             | No                                                  | 'USD'             |

For example:

```javascript
snowplow_name_here('addItem',
    '1234',           // order ID - required
    'DD44',           // SKU/code - required
    'T-Shirt',        // product name
    'Green Medium',   // category or variation
    '11.99',          // unit price - required
    '1',              // quantity - required
    'USD'             // currency
  );
```

`addItem` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

<a name="trackTrans" />

#### 3.3.3 `trackTrans`

Once the transaction object has been created (using `addTrans`) and the relevant item data added to it using the `addItem` method, we are ready to send the data to the collector. This is initiated using the `trackTrans` method:

```javascript
snowplow_name_here('trackTrans');
```

<a name="ecomm-example" />

#### 3.3.4 Putting the three methods together: a complete example

```html
<html>
<head>
<title>Receipt for your clothing purchase from Acme Clothing</title>
<script type="text/javascript">

  ;(function(p,l,o,w,i,n,g){if(!p[i]){p.GlobalSnowplowNamespace=p.GlobalSnowplowNamespace||[];
  p.GlobalSnowplowNamespace.push(i);p[i]=function(){(p[i].q=p[i].q||[]).push(arguments)
  };p[i].q=p[i].q||[];n=l.createElement(o);g=l.getElementsByTagName(o)[0];n.async=1;
  n.src=w;g.parentNode.insertBefore(n,g)}}(window,document,"script","//d1fc8wv8zag5ca.cloudfront.net/2.5.1/sp.js","snowplow_name_here"));

  snowplow_name_here('newTracker', 'cf', 'd3rkrsqld9gmqf.cloudfront.net');
  snowplow_name_here('enableActivityTracking', 30, 10)
  snowplow_name_here('trackPageView');
  snowplow_name_here('enableLinkClickTracking');

  snowplow_name_here('addTrans',
    '1234',           // order ID - required
    'Acme Clothing',  // affiliation or store name
    '11.99',          // total - required
    '1.29',           // tax
    '5',              // shipping
    'San Jose',       // city
    'California',     // state or province
    'USA'             // country
  );

   // add item might be called for every item in the shopping cart
   // where your ecommerce engine loops through each item in the cart and
   // prints out _addItem for each
  snowplow_name_here('addItem',
    '1234',           // order ID - required
    'DD44',           // SKU/code - required
    'T-Shirt',        // product name
    'Green Medium',   // category or variation
    '11.99',          // unit price - required
    '1'               // quantity - required
  );

  // trackTrans sends the transaction to Snowplow tracking servers.
  // Must be called last to commit the transaction.
  snowplow_name_here('trackTrans'); //submits transaction to the collector

</script>
</head>
<body>

  Thank you for your order.  You will receive an email containing all your order details.

</body>
</html>
```

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="social" />

### 3.4 Social tracking

Social tracking will be used to track the way users interact with Facebook, Twitter and Google + widgets, e.g. to capture "like this" or "tweet this" events.

<a name="trackSocial" />

#### 3.4.1 `trackSocialInteraction`

The `trackSocialInteraction` method takes three parameters:

| **Parameter** | **Description**                                               | **Required?** | **Example value**     |
|:--------------|:--------------------------------------------------------------|:--------------|:----------------------|
| `action`      | Social action performed                                       | Yes           | 'like', 'retweet'     |
| `network`     | Social network                                                | Yes           | 'facebook', 'twitter' |
| `target`      | Object social action is performed on e.g. page ID, product ID | No            | '19.99'               |

The method is executed in as:

```javascript
snowplow_name_here('trackSocialInteraction', action, network, target);
```

For example:

```javascript
snowplow_name_here('trackSocialInteraction', 'like', 'facebook', 'pbz00123');
```

Or if the optional parameters were left off:

```javascript
snowplow_name_here('trackSocialInteraction', 'like', 'facebook');
```

`trackSocialInteraction` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="campaign" />

### 3.5 Campaign tracking

Campaign tracking is used to identify the source of traffic coming to a website.

At the highest level, we can distinguish **paid** traffic (that derives from ad spend) with **non paid** traffic: visitors who come to the website by entering the URL directly, clicking on a link from a referrer site or clicking on an organic link returned in a search results, for example.

In order to identify **paid** traffic, Snowplow users need to set five query parameters on the links used in ads. Snowplow checks for the presence of these query parameters on the web pages that users load: if it finds them, it knows that that user came from a paid source, and stores the values of those parameters so that it is possible to identify the paid source of traffic exactly.

If the query parameters are not present, Snowplow reasons that the user is from a **non paid** source of traffic. It then checks the page referrer (the url of the web page the user was on before visiting our website), and uses that to deduce the source of traffic:

1. If the URL is identified as a search engine, the traffic medium is set to "organic" and Snowplow tries to derive the search engine name from the referrer URL domain and the keywords from the  query string.
2. If the URL is a non-search 3rd party website, the medium is set to "referrer". Snowplow derives the source from the referrer URL domain.

<a name="identifying-paid-sources" />

#### 3.5.1 Identifying paid sources

Your different ad campaigns (PPC campaigns, display ads, email marketing messages, Facebook campaigns etc.) will include one or more links to your website e.g.:

```html
<a href="http://mysite.com/myproduct.html">Visit website</a>
```

We want to be able to identify people who've clicked on ads e.g. in a marketing email as having come to the site having clicked on a link in that particular marketing email. To do that, we modify the link in the marketing email with query parameters, like so:

```html
<a href="http://mysite.com/myproduct.html?utm_source=newsletter-october&utm_medium=email&utm_campaign=cn0201">Visit website</a>
```

For the prospective customer clicking on the link, adding the query parameters does not change the user experience. (The user is still directed to the webpage at http://mysite.com/myproduct.html.) But Snowplow then has access to the fields given in the query string, and uses them to identify this user as originating from the October Newsletter, an email marketing campaign with campaign id = cn0201.

<a name="anatomy-of-the-query-parameters" />

#### 3.5.2 Anatomy of the query parameters

Snowplow uses the same query parameters used by Google Analytics. Because of this, Snowplow users who are also using GA do not need to do any additional work to make their campaigns trackable in Snowplow as well as GA. Those parameters are:

| **Parameter**  |     **Name**     |                                                                    **Description**                                                                    |
|:--------------:|:----------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------:|
|  `utm_source`  | Campaign source  |                          Identify the advertiser driving traffic to your site e.g. Google, Facebook, autumn-newsletter etc.                           |
|  `utm_medium`  | Campaign medium  |                                 The advertising / marketing medium e.g. cpc, banner, email newsletter, in-app ad, cpa                                 |
| `utm_campaign` |   Campaign id    |   A unique campaign id. This can be a descriptive name or a number / string that is then looked up against a campaign table as part of the analysis   |
|  `utm_term  `  | Campaign term(s) | Used for search marketing in particular, this field is used to identify the search terms that triggered the ad being displayed in the search results. |
| `utm_content`  | Campaign content |   Used either to differentiate similar content or two links in the same ad. (So that it is possible to identify which is generating more traffic.)    |

The parameters are descibed in the [Google Analytics help page][gahelppage]. Google also provides a [urlbuilder][gaurlbuilder] which can be used to construct the URL incl. query parameters to use in your campaigns.


[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="ad-tracking" />

### 3.6 Ad tracking methods

Snowplow tracking code can be included in ad tags in order to track impressions and ad clicks. This is used by e.g. ad networks to identify which sites and web pages users visit across a network, so that they can be segmented, for example.

Each ad tracking method has a `costModel` field and a `cost` field. If you provide the `cost` field, you must also provide one of `'cpa'`, `'cpc'`, and `'cpm'` for the `costModel` field.

It may be the case that multiple ads from the same source end up on a single page. If this happens, it is important that the different Snowplow code snippets associated with those ads not interfere with one another. The best way to prevent this is to randomly name each tracker instance you create so that the probability of a name collision is negligible. See [Managing multiple trackers](1-General-parameters-for-the-Javascript-tracker#25-managing-multiple-trackers) for more on having more than one tracker instance on a single page.

Below is an example of how to achieve this when using Snowplow ad impression tracking.

```html
<!-- Snowplow starts plowing -->
<script type="text/javascript">

// Wrap script in a closure.
// This prevents rnd from becoming a global variable.
// So if multiple copies of the script are loaded on the same page,
// each instance of rnd will be inside its own namespace and will
// not overwrite any of the others.
// See http://benalman.com/news/2010/11/immediately-invoked-function-expression/
(function(){
  // Randomly generate tracker namespace to prevent clashes
  var rnd = Math.random().toString(36).substring(2);

  // Load Snowplow
  ;(function(p,l,o,w,i,n,g){if(!p[i]){p.GlobalSnowplowNamespace=p.GlobalSnowplowNamespace||[];
  p.GlobalSnowplowNamespace.push(i);p[i]=function(){(p[i].q=p[i].q||[]).push(arguments)
  };p[i].q=p[i].q||[];n=l.createElement(o);g=l.getElementsByTagName(o)[0];n.async=1;
  n.src=w;g.parentNode.insertBefore(n,g)}}(window,document,"script","//d1fc8wv8zag5ca.cloudfront.net/2.5.1/sp.js","sp_pm"));

  // Create a new tracker namespaced to rnd
  window.sp_pm('newTracker', rnd, 'dgrp31ac2azr9.cloudfront.net', {
    appId: 'myApp',
    platform: 'web'
  });

  // Replace the values below with magic macros from your adserver
  window.sp_pm('trackAdImpression:' + rnd,
    '67965967893',            // impressionId
    'cpm',                    // costModel - 'cpa', 'cpc', or 'cpm'
    5.5,                      // cost
    'http://www.example.com', // targetUrl
    '23',                     // bannerId
    '7',                      // zoneId
    '201',                    // advertiserId
    '12'                      // campaignId
  );
}());
</script>
<!-- Snowplow stops plowing -->
```

Even if several copies of the above script appear on a page, the trackers created will all (probably) have different names and so will not interfere with one another. The same technique should be used when tracking ad clicks. The below examples for `trackAdImpression` and `trackAdClick` assume that `rnd` has been defined in this way.

<a name="adImpression" />

#### 3.6.1 `trackAdImpression`

Ad impression tracking is accomplished using the `trackAdImpression` method. Here are the arguments it accepts:

|       **Name** | **Required?** | **Description**                                                      | **Type** |
|---------------:|:--------------|:---------------------------------------------------------------------|:---------|
| `impressionId` | No            | Identifier for the particular impression instance                    | string   |
|    `costModel` | No            | The cost model for the campaign: 'cpc', 'cpm', or 'cpa'              | string   |
|         `cost` | No            | Ad cost                                                              | number   |
|    `targetUrl` | No            | The destination URL                                                  | string   |
|     `bannerId` | No            | Adserver identifier for the ad banner (creative) being displayed     | string   |
|       `zoneId` | No            | Adserver identifier for the zone where the ad banner is located      | string   |
| `advertiserID` | No            | Adserver identifier for the advertiser which the campaign belongs to | string   |
|   `campaignId` | No            | Adserver identifier for the ad campaign which the banner belongs to  | string   |

An example:

```javascript
snowplow_name_here('trackAdImpression:' + rnd,

    '67965967893',             // impressionId
    'cpm',                     // costModel - 'cpa', 'cpc', or 'cpm'
     5.5,                      // cost
    'http://www.example.com',  // targetUrl
    '23',                      // bannerId
    '7',                       // zoneId
    '201',                     // advertiserId
    '12'                       // campaignId
);
```

Ad impression events are implemented as Snowplow unstructured events. [Here][ad-impression-schema] is the JSON schema for an ad impression event.

`trackAdImpression` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

You will want to set these arguments programmatically, across all of your ad zones/slots. For guidelines on how to achieve this with the [OpenX adserver][openx], please see the following section [3.6.4](#ad-example).

<a name="adClick" />

#### 3.6.2 `trackAdClick`

Ad click tracking is accomplished using the `trackAdClick` method. Here are the arguments it accepts:

|       **Name** | **Required?** | **Description**                                                      | **Type** |
|---------------:|:--------------|:---------------------------------------------------------------------|:---------|
|    `targetUrl` | Yes           | The destination URL                                                  | string   |
|      `clickId` | No            | Identifier for the particular click instance                         | string   |
|    `costModel` | No            | The cost model for the campaign: 'cpc', 'cpm', or 'cpa'              | string   |
|         `cost` | No            | Ad cost                                                              | number   |
|     `bannerId` | No            | Adserver identifier for the ad banner (creative) being displayed     | string   |
|       `zoneId` | No            | Adserver identifier for the zone where the ad banner is located      | string   |
| `advertiserID` | No            | Adserver identifier for the advertiser which the campaign belongs to | string   |
|   `campaignId` | No            | Adserver identifier for the ad campaign which the banner belongs to  | string   |

An example:

```javascript
snowplow_name_here('trackAdClick:' + rnd,

    'http://www.example.com',  // targetUrl
    '12243253',                // clickId
    'cpm',                     // costModel
     2.5,                      // cost
    '23',                      // bannerId
    '7',                       // zoneId
    '67965967893',             // impressionId - the same as in trackAdImpression
    '201',                     // advertiserId
    '12'                       // campaignId
);
```

Ad click events are implemented as Snowplow unstructured events.[Here][ad-click-schema] is the JSON schema for an ad click event.

`trackAdClick` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

<a name="adConversion" />

#### 3.6.3 `trackAdConversion`

Use the `trackAdConversion` method to track ad conversions.  Here are the arguments it accepts:

|       **Name** | **Required?** | **Description**                                                      | **Type** |
|---------------:|:--------------|:---------------------------------------------------------------------|:---------|
| `conversionId` | No            | Identifier for the particular conversion instance                    | string   |
|    `costModel` | No            | The cost model for the campaign: 'cpc', 'cpm', or 'cpa'              | string   |
|         `cost` | No            | Ad cost                                                              | number   |
|     `category` | No            | Conversion category                                                  | number   |
|       `action` | No            | The type of user interaction, e.g. 'purchase'                        | string   |
|     `property` | No            | Describes the object of the conversion                               | string   |
| `initialValue` | No            | How much the conversion is initially worth                           | number   |
| `advertiserID` | No            | Adserver identifier for the advertiser which the campaign belongs to | string   |
|   `campaignId` | No            | Adserver identifier for the ad campaign which the banner belongs to  | string   |

An example:

```javascript
window.snowplow_name_here('trackAdConversion',

    '743560297', // conversionId
    10,          // cost
    'ecommerce', // category
    'purchase',  // action
    '',          // property
    99,          // initialValue - how much the conversion is initially worth
    '201',       // advertiserId
    'cpa',       // costModel
    '12'         // campaignId
);
```

Ad conversion events are implemented as Snowplow unstructured events. [Here][ad-conversion-schema] is the schema for an ad conversion event.

`trackAdConversion` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

<a name="ad-example" />

#### 3.6.4 Example: implementing impression tracking with Snowplow and OpenX

Most ad servers enable you to append custom code to your ad tags. Here's what the zone append functionality looks like in the OpenX adserver (OnRamp edition):

![zoneappend][zoneappend]

You will need to populate the ad zone append field with Snowplow tags for **every ad zone/unit** which you use to serve ads across your site or network. Read on for the Snowplow HTML code to use for OpenX.

#### OpenX: Snowplow impression tracking using magic macros

Because OpenX has a feature called [magic macros][magicmacros], it is relatively straightforward to pass the banner, campaign and user ID arguments into the call to `trackAdImpression()` (advertiser ID is not available through magic macros).

The full HTML code to append, using asynchronous Snowplow invocation, looks like this:

```html
<!-- Snowplow starts plowing -->
<script type="text/javascript">
;(function(p,l,o,w,i,n,g){if(!p[i]){p.GlobalSnowplowNamespace=p.GlobalSnowplowNamespace||[];
p.GlobalSnowplowNamespace.push(i);p[i]=function(){(p[i].q=p[i].q||[]).push(arguments)
};p[i].q=p[i].q||[];n=l.createElement(o);g=l.getElementsByTagName(o)[0];n.async=1;
n.src=w;g.parentNode.insertBefore(n,g)}}(window,document,"script","//d1fc8wv8zag5ca.cloudfront.net/2.5.1/sp.js","snowplow_name_here"));

// Update tracker constructor to use your CloudFront distribution subdomain
window.snowplow_name_here('newTracker', 'cf', 'patldfvsg0d8w.cloudfront.net');

window.snowplow_name_here('trackAdImpression', '{impressionId}' '{costModel}', '{cost}', '{targetUrl}', '{bannerid}', '{zoneid}', '{advertiserId}', '{campaignid}'); // OpenX magic macros. Leave this line as-is

 </script>
<!-- Snowplow stops plowing -->
```

Once you have appended this code to all of your active ad zones, Snowplow should be collecting all of your ad impression data.

[Back to top](#top)

<a name="custom-structured-events" />

### 3.7 Tracking custom structured events

There are likely to be a large number of AJAX events that can occur on your site, for which a specific tracking method is part of Snowplow. Examples include:

* Playing a video
* Adding an item to basket
* Submitting a lead form

Our philosophy in creating Snowplow is that users should capture "every" consumer interaction and work out later how to use this data. This is different from traditional web analytics and business intelligence, that argues that you should first work out what you need, and only then start capturing the data.

As part of a Snowplow implementation, therefore, we recommend that you identify every type of AJAX interaction that a user might have with your site: each one of these is an event that will not be captured as part of the standard page view tracking. All of them are candidates to track using `trackStructEvent`, if none of the other event-specific methods outlined above are appropriate.

<a name="trackStructEvent" />

#### 3.7.1 `trackStructEvent`

There are five parameters can be associated with each structured event. Of them, only the first two are required:

|   **Name** | **Required?** | **Description**                                                                                                                                                                      |
|-----------:|:--------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Category` | Yes           | The name you supply for the group of objects you want to track e.g. 'media', 'ecomm'                                                                                                 |
|   `Action` | Yes           | A string which defines the type of user interaction for the web object e.g. 'play-video', 'add-to-basket'                                                                            |
|    `Label` | No            | An optional string which identifies the specific object being actioned e.g. ID of the video being played, or the SKU or the product added-to-basket                                  |
| `Property` | No            | An optional string describing the object or the action performed on it. This might be the quantity of an item added to basket                                                        |
|    `Value` | No            | An optional float to quantify or further describe the user action. This might be the price of an item added-to-basket, or the starting time of the video where play was just pressed |

The async specification for the `trackStructEvent` method is:

```javascript
snowplow_name_here('trackStructEvent', 'category','action','label','property','value');
```

An example of tracking a user listening to a music mix:

```javascript
snowplow_name_here('trackStructEvent', 'Mixes', 'Play', 'MrC/fabric-0503-mix', '', '0.0');
```

Note that in the above example no value is set for the `event property`.

`trackStructEvent` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="custom-selfdescribing-events" />

### 3.8 Tracking custom self-describing (unstructured) events

*Note that this is only available on version 2.7.0 and above of the JS tracker.*

You may wish to track events on your website or application which are not directly supported by Snowplow and which [structured event tracking](#custom-structured-events) does not adequately capture.
Your event may have more than the five fields offered by `trackStructEvent`, or its fields may not fit into the category-action-label-property-value model.
The solution is Snowplow's self-describing events (previously known as unstructured). Self-describing events use JSONs which can have arbitrarily many fields.

To define your own custom event, you must create a [JSON schema][json-schema] for that event and upload it to an [Iglu Schema Repository][iglu-repo].
Snowplow uses the schema to validate that the JSON containing the event properties is well-formed.

<a name="trackSelfDescribingEvent" />

#### 3.8.1 `trackSelfDescribingEvent`

To track a self-describing event, you make use the `trackSelfDescribingEvent` method:

```javascript
snowplow_name_here('trackSelfDescribingEvent', <<SELF-DESCRIBING EVENT JSON>>);
```

For example:

```javascript
window.snowplow_name_here('trackSelfDescribingEvent', {
    schema: 'iglu:com.acme_company/viewed_product/jsonschema/2-0-0',
    data: {
        productId: 'ASO01043',
        category: 'Dresses',
        brand: 'ACME',
        returning: true,
        price: 49.95,
        sizes: ['xs', 's', 'l', 'xl', 'xxl'],
        availableSince: new Date(2013,3,7)
    }
});
```

The second argument is a [self-describing JSON][self-describing-jsons]. It has two fields:

* A `data` field, containing the properties of the event
* A `schema` field, containing the location of the [JSON schema][json-schema] against which the `data` field should be validated.

The `data` field should be flat, not nested.

`trackSelfDescribingEvent` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="link-click-tracking" />

### 3.9 Link click tracking

Link click tracking is enabled using the `enableLinkClickTracking` method. Use this method once and the Tracker will add click event listeners to all link elements. Link clicks are tracked as unstructured events. Each link click event captures the link's href attribute. The event also has fields for the link's id, classes, and target (where the linked document is opened, such as a new tab or new window).

[Here][link-click-schema] is the JSON schema for a link click event.

<a name="enableLinkClickTracking" />

#### 3.9.1 `enableLinkClickTracking`

Turn on link click tracking like this:

```javascript
snowplow_name_here('enableLinkClickTracking');
```

This is its signature:

```javascript
enableLinkClickTracking(filter, pseudoclicks, content);
```

You can control which links are tracked using the second argument. There are three ways to do this: a blacklist, a whitelist, and a filter function.

__Blacklists__

This is an array of CSS classes which should be ignored by link click tracking. For example, the below code will stop link click events firing for links with the class "barred" or "untracked", but will fire link click events for all other links:

```javascript
snowplow_name_here('enableLinkClickTracking', {'blacklist': ['barred', 'untracked']});
```

If there is only one class name you wish to blacklist, you don't need to put it in an array:

```javascript
snowplow_name_here('enableLinkClickTracking', {'blacklist': 'barred'});
```

 __Whitelists__

The opposite of a blacklist. This is an array of the CSS classes of links which you do want to be tracked. Only clicks on links with a class in the list will be tracked.

```javascript
snowplow_name_here('enableLinkClickTracking', {'whitelist': ['unbarred', 'tracked']});
```

If there is only one class name you wish to whitelist, you don't need to put it in an array:

```javascript
snowplow_name_here('enableLinkClickTracking', {'whitelist': 'unbarred'});
```

__Filter functions__

You can provide a filter function which determines which links should be tracked. The function should take one argument, the link element, and return either 'true' (in which case clicks on the link will be tracked) or 'false' (in which case they won't).

The following code will track clicks on those and only those links whose id contains the string "interesting":

```javascript
function myFilter (linkElement) {
  return linkElement.id.indexOf('interesting') > -1;
}

snowplow_name_here('enableLinkClickTracking', {'filter': myFilter});
```

The second optional parameter is `pseudoClicks`. If this is not turned on, Firefox will not recognise middle clicks. If it is turned on, there is a small possibility of false positives (click events firing when they shouldn't). Turning this feature on is recommended:

```javascript
snowplow_name_here('enableLinkClickTracking', null, true);
```

The third optional parameter is `content`. Set it to `true` if you want link click events to capture the innerHTML of the clicked link:

```javascript
snowplow_name_here('enableLinkClickTracking', null, null, true);
```

The innerHTML of a link is all the text between the `a` tags. Note that if you use a base 64 encoded image as a link, the entire base 64 string will be included in the event.

Each link click event will include (if available) the destination URL, id, classes and target of the clicked link. (The target attribute of a link specifies a window or frame where the linked document will be loaded.)

`enableLinkClickTracking` can also be passed an array of custom contexts to attach to every link click event as an additional final parameter. See [Contexts](#custom-contexts) for more information.

<a name="refreshLinkClickTracking" />

#### 3.9.2 `refreshLinkClickTracking`

`enableLinkClickTracking` only tracks clicks on links which exist when the page has loaded. If new links can be added to the page after then which you wish to track, just use `refreshLinkClickTracking`. This will add Snowplow click listeners to all links which do not already have them (and which match the blacklist, whitelist, or filter function you specified when `enableLinkClickTracking` was originally called). Use it like this:

```javascript
snowplow_name_here('refreshLinkClickTracking');
```

<a name="trackLinkClick" />

#### 3.9.3 `trackLinkClick`

You can manually track individual link click events with the `trackLinkClick` method. This is its signature:

```javascript
function trackLinkClick(targetUrl, elementId, elementClasses, elementTarget, elementContent);
```

Of these arguments, only `targetUrl` is required. This is how to use `trackLinkClick`:

```javascript
snowplow_name_here('trackLinkClick', 'http://www.example.com', 'first-link', ['class-1', 'class-2'], '', 'this page');
```

`trackLinkClick` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="form-tracking" />

### 3.10 Form tracking

Snowplow automatic form tracking detects two event types:

##### `change_form`

When a user changes the value of a `textarea`, `input`, or `select` element inside a form, a [`change_form`][change_form] event will be fired. It will capture the name, type, and new value of the element, and the id of the parent form.

##### `submit_form`

When a user submits a form, a [`submit_form`][submit_form] event will be fired. It will capture the id and classes of the form and the name, type, and value of all `textarea`, `input`, and `select` elements inside the form.

Note that this will only work if the original form submission event is actually fired. If you prevent it from firing, for example by using a jQuery event handler which returns `false` to handle clicks on the form's submission button, the Snowplow `submit_form` event will not be fired.

<a name="enableFormTracking" />

#### 3.10.1 `enableFormTracking`

Use the `enableFormTracking` method to add event listeners to turn on form tracking by adding event listeners to all form elements and to all interactive elements inside forms (that is, all `input`, `textarea`, and `select` elements).

```javascript
snowplow('enableFormTracking');
```

This will only work for form elements which exist when it is called. If you are creating a form programatically, call `enableFormTracking` again after adding it to the document to track it. (You can call `enableFormTracking` multiple times without risk of duplicated events.)

Note that events on password fields will not be tracked.

<a name="custom-form-tracking" />

#### 3.10.2 Custom form tracking

It may be that you do not want to track every field in a form, or every form on a page. You can customize form tracking by passing a configuration argument to the `enableFormTracking` method. This argument should be an object with two elements named "forms" and "fields". The "forms" element determines which forms will be tracked; the "fields" element determines which fields inside the tracked forms will be tracked. As with link click tracking, there are three ways to configure each field: a blacklist, a whitelist, or a filter function. You do not have to use the same method for both fields.

__Blacklists__

This is an array of strings used to prevent certain elements from being tracked. Any form with a CSS class in the array will be ignored. Any field whose name property is in the array will be ignored. All other elements will be tracked.

__Whitelists__

This is an array of strings used to turn on certail. Any form with a CSS class in the array will be tracked. Any field in a tracked form whose name property is in the array will be tracked. All other elements will be ignored.

__Filter functions__

This is a function used to determine which elements are tracked. The element is passed as the argument to the function and is tracked if and only if the value returned by the function is truthy.

__Transform functions__

This is a function used to transform data in each form field. The element is passed as the argument to the function and is replaced by the value returned.

__Examples__

To track every form element and every field except those fields named "password":

```javascript
var config = {
  forms: {
    blacklist: []
  },
  fields: {
    blacklist: ['password']
  }
};

snowplow('enableFormTracking', config);
```

To track only the forms with CSS class "tracked", and only those fields whose ID is not "private":

```javascript
var config = {
  forms: {
    whitelist: ["tracked"]
  },
  fields: {
    filter: function (elt) {
      return elt.id !== "private";
    }
  }
};

snowplow('enableFormTracking', config);
```

To transform the form fields with an MD5 hashing function:

```
var config = {
  forms: {
    whitelist: ["tracked"]
  },
  fields: {
    filter: function (elt) {
      return elt.id !== "private";
    },
    transform: function (elt) {
      return MD5(elt);
    }
  }
};

snowplow('enableFormTracking', config);
```

<a name="cart" />

### 3.11 `trackAddToCart` and `trackRemoveFromCart`

These methods let you track users adding and removing items from a cart on an ecommerce site. Their arguments are identical:

|    **Name** | **Required?** | **Description**                        | **Type** |
|------------:|:--------------|:---------------------------------------|:---------|
|       `sku` | Yes           | Item SKU                               | string   |
|      `name` | No            | Item name                              | string   |
|  `category` | No            | Item category                          | string   |
| `unitPrice` | Yes           | Item price                             | number   |
|  `quantity` | Yes           | Quantity added to or removed from cart | number   |
|  `currency` | No            | Item price currency                    | string   |

An example:

```javascript
window.snowplow_name_here('trackAddToCart', '000345', 'blue tie', 'clothing', 3.49, 2, 'GBP');
window.snowplow_name_here('trackRemoveFromCart', '000345', 'blue tie', 'clothing', 3.49, 1, 'GBP');
```

Both methods are implemented as Snowplow unstructured events. You can see schemas for the [`add_to_cart`][add_to_cart] and [`remove_from_cart`][remove_from_cart] events.

Both methods can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

<a name="siteSearch" />

### 3.12 `trackSiteSearch`

Use the `trackSiteSearch` method to track users searching your website. Here are its arguments:

|       **Name** | **Required?** | **Description**                 | **Type** |
|---------------:|:--------------|:--------------------------------|:---------|
|        `terms` | Yes           | Search terms                    | array    |
|      `filters` | No            | Search filters                  | JSON     |
| `totalResults` | No            | Results found                   | number   |
|  `pageResults` | No            | Results displayed on first page | number   |

An example:

```javascript
window.snowplow_name_here('trackSiteSearch',
    ['unified', 'log'],      // search terms
    {'category': 'books', 'sub-category': 'non-fiction'},  // filters
    14,                      // results found
    8                        // results displayed on first page
);
```

Site search events are implemented as Snowplow unstructured events. [Here][site_search] is the schema for a `site_search` event.

`trackSiteSearch` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

<a name="timing" />

### 3.13 `trackTiming`

Use the `trackTiming` method to track user timing events such as how long resources take to load. Here are its arguments:

|   **Name** | **Required?** | **Description**                | **Type** |
|-----------:|:--------------|:-------------------------------|:---------|
| `category` | Yes           | Timing category                | string   |
| `variable` | Yes           | Timed variable                 | string   |
|   `timing` | Yes           | Number of milliseconds elapsed | number   |
|    `label` | No            | Label for the event            | string   |

An example:

```javascript
window.snowplow('trackTiming',
  'load',            // Category of the timing variable
  'map_loaded',      // Variable being recorded
  50,                // Milliseconds taken
  'Map loading time' // Optional label
);
```

Site search events are implemented as Snowplow unstructured events. [Here][timing] is the schema for a `timing` event.

`trackTiming` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

<a name="enhanced-ecommerce" />

### 3.14 Enhanced Ecommerce tracking

For more information on the Enhanced Ecommerce functions please see the Google Analytics [documentation](https://developers.google.com/analytics/devguides/collection/analyticsjs/enhanced-ecommerce).

<a name="addEnhancedEcommerceActionContext" />

#### 3.14.1 `addEnhancedEcommerceActionContext`

Use the `addEnhancedEcommerceActionContext` method to add a GA Enhanced Ecommerce Action Context to the Tracker:

|      **Name** | **Required?** | **Type**          |
|--------------:|:--------------|:------------------|
|          `id` | Yes           | string            |
| `affiliation` | No            | string            |
|     `revenue` | No            | number OR string  |
|         `tax` | No            | number OR string  |
|    `shipping` | No            | number OR string  |
|      `coupon` | No            | string            |
|        `list` | No            | string            |
|        `step` | No            | integer OR string |
|      `option` | No            | string            |
|    `currency` | No            | string            |

Adding an action using Google Analytics:

```javascript
ga('ec:setAction', 'purchase', {
  'id': 'T12345',
  'affiliation': 'Google Store - Online',
  'revenue': '37.39',
  'tax': '2.85',
  'shipping': '5.34',
  'coupon': 'SUMMER2013'
});
```

__NOTE__: The action type is passed with the action context in the Google Analytics example.  We have seperated this by asking you to call the [trackEnhancedEcommerceAction](#trackEnhancedEcommerceAction) function to actually send the context and the action.

Adding an action using Snowplow:

```javascript
window.snowplow('addEnhancedEcommerceActionContext',
  'T12345', // The Transaction ID
  'Google Store - Online', // The affiliate
  '37.39', // The revenue
  '2.85', // The tax
  '5.34', // The shipping
  'WINTER2016' // The coupon
);
```

<a name="addEnhancedEcommerceImpressionContext" />

#### 3.14.2 `addEnhancedEcommerceImpressionContext`

Use the `addEnhancedEcommerceImpressionContext` method to add a GA Enhanced Ecommerce Impression Context to the Tracker:

|   **Name** | **Required?** | **Type**          |
|-----------:|:--------------|:------------------|
|       `id` | Yes           | string            |
|     `name` | No            | string            |
|     `list` | No            | string            |
|    `brand` | No            | string            |
| `category` | No            | string            |
|  `variant` | No            | string            |
| `position` | No            | integer OR string |
|    `price` | No            | number OR string  |
| `currency` | No            | string            |

Adding an impression using Google Analytics:

```javascript
ga('ec:addImpression', {
  'id': 'P12345',
  'name': 'Android Warhol T-Shirt',
  'list': 'Search Results',
  'brand': 'Google',
  'category': 'Apparel/T-Shirts',
  'variant': 'Black',
  'position': 1
});
```

Adding an impression using Snowplow:

```javascript
window.snowplow('addEnhancedEcommerceImpressionContext',
  'P12345', // The ID
  'Android Warhol T-Shirt', // The name
  'Search Results', // The list
  'Google', // The brand
  'Apparel/T-Shirts', // The category
  'Black', // The variant
  '1' // The position
);
```

<a name="addEnhancedEcommerceProductContext" />

#### 3.14.3 `addEnhancedEcommerceProductContext`

Use the `addEnhancedEcommerceProductContext` method to add a GA Enhanced Ecommerce Product Field Context:

|   **Name** | **Required?** | **Type**          |
|-----------:|:--------------|:------------------|
|       `id` | Yes           | string            |
|     `name` | No            | string            |
|     `list` | No            | string            |
|    `brand` | No            | string            |
| `category` | No            | string            |
|  `variant` | No            | string            |
|    `price` | No            | number OR string  |
| `quantity` | No            | integer OR string |
|   `coupon` | No            | string            |
| `position` | No            | integer OR string |
| `currency` | No            | string            |

Adding a product using Google Analytics:

```javascript
ga('ec:addProduct', {
  'id': 'P12345',
  'name': 'Android Warhol T-Shirt',
  'brand': 'Google',
  'category': 'Apparel/T-Shirts',
  'variant': 'Black',
  'position': 1
});
```

Adding a product using Snowplow:

```javascript
window.snowplow('addEnhancedEcommerceProductContext',
  'P12345', // The ID
  'Android Warhol T-Shirt', // The name
  'Search Results', // The list
  'Google', // The brand
  'Apparel/T-Shirts', // The category
  'Black', // The variant
  1 // The quantity
);
```

<a name="addEnhancedEcommercePromoContext" />

#### 3.14.4 `addEnhancedEcommercePromoContext`

Use the `addEnhancedEcommercePromoContext` method to add a GA Enhanced Ecommerce Promotion Field Context:

|   **Name** | **Required?** | **Type** |
|-----------:|:--------------|:---------|
|       `id` | Yes           | string   |
|     `name` | No            | string   |
| `creative` | No            | string   |
| `position` | No            | string   |
| `currency` | No            | string   |

Adding a promotion using Google Analytics:

```javascript
ga('ec:addPromo', {
  'id': 'PROMO_1234',
  'name': 'Summer Sale',
  'creative': 'summer_banner2',
  'position': 'banner_slot1'
});
```

Adding a promotion using Snowplow:

```javascript
window.snowplow('addEnhancedEcommercePromoContext',
  'PROMO_1234', // The Promotion ID
  'Summer Sale', // The name
  'summer_banner2', // The name of the creative
  'banner_slot1' // The position
);
```

<a name="trackEnhancedEcommerceAction" />

#### 3.14.5 `trackEnhancedEcommerceAction`

Use the `trackEnhancedEcommerceAction` method to track a GA Enhanced Ecommerce Action.  When this function is called all of the added Ecommerce Contexts are attached to this action and flushed from the Tracker.

| **Name** | **Required?** | **Type** |
|---------:|:--------------|:---------|
| `action` | Yes           | string   |

The allowed actions:

 * `click`
 * `detail`
 * `add`
 * `remove`
 * `checkout`
 * `checkout_option`
 * `purchase`
 * `refund`
 * `promo_click`
 * `view`

Adding an action using Google Analytics:

```javascript
ga('ec:setAction', 'refund', {
  'id': 'T12345'
});
```

Adding an action using Snowplow:

```javascript
window.snowplow('addEnhancedEcommerceActionContext',
  'T12345' // ID
);
window.snowplow('trackEnhancedEcommerceAction',
  'refund'
);
```

<a name="consent" />

### 3.15 Consent tracking

<a name="trackConsentGranted" />

#### 3.15.1 `trackConsentGranted`
Use the `trackConsentGranted` method to track a user opting into data collection. A consent document context will be attached to the event if at least the `id` and `version` arguments are supplied. The method arguments are:

|      **Name** | **Description**                                           | **Required?** | **Type**         |
|--------------:|:----------------------------------------------------------|:--------------|:-----------------|
|          `id` | Identifier for the document granting consent              | Yes           | Number           |
|     `version` | Version of the document granting consent                  | Yes           | Number           |
|        `name` | Name of the document granting consent                     | No            | String           |
| `description` | Description of the document granting consent              | No            | String           |
|      `expiry` | Date-time string specifying when consent document expires | No            | String           |
|     `context` | Custom context for the event                              | No            | Array            |
|      `tstamp` | When the event occurred                                   | No            | Positive integer |

The `expiry` field specifies that the user consents to the attached documents until the date-time provided, after which the consent is no longer valid.

Tracking a consent granted event:

```javascript
window.snowplow('trackConsentGranted',
  1234,                          // Id
  5,                             // Version
  'consent_document',            // Name
  'a document granting consent', // Description
  '2020-11-21T08:00:00.000Z'     // Expiry
);
```

<a name="trackConsentWithdrawn" />

#### 3.15.2 `trackConsentWithdrawn`
Use the `trackConsentWithdrawn` method to track a user withdrawing consent for data collection. A consent document context will be attached to the event if at least the `id` and `version` arguments are supplied. To specify that a user opts out of all data collection, `all` should be set to `true`.

The method arguments are:

|      **Name** | **Description**                                   | **Required?** | **Type**         |
|--------------:|:--------------------------------------------------|:--------------|:-----------------|
|          `id` | Identifier for the document withdrawing consent   | No            | String           |
|     `version` | Version of the document withdrawing consent       | No            | string           |
|        `name` | Name of the document withdrawing consent          | No            | String           |
| `description` | Description of the document withdrawing consent   | No            | String           |
|         `all` | Specifies whether all consent should be withdrawn | No            | Boolean          |
|     `context` | Custom context for the event                      | No            | Array            |
|      `tstamp` | When the event occurred                           | No            | Positive integer |

Tracking a consent withdrawn event:

```javascript
window.snowplow('trackConsentWithdrawn',
  1234,                             // Id
  5,                                // Version
  'consent_document',               // Name
  'a document withdrawing consent', // Description
  'false'                           // All
);
```

<a name="consent-documents" />

#### 3.15.3 Consent documents
Consent documents are stored in the context of a consent event. Each consent method adds a consent document to the event. The consent document is a custom context storing the arguments supplied to the method (in both granted and withdrawn events, this will be: id, version, name, and description). In either consent method, additional documents can be appended to the event by passing an array of consent document self-describing JSONs in the context argument.

The fields of a consent document are:

|      **Name** | **Description**             | **Required?** | **Type** |
|--------------:|:----------------------------|:--------------|:---------|
|          `id` | Identifier for the document | Yes           | Number   |
|     `version` | Version of the document     | Yes           | Number   |
|        `name` | Name of the document        | No            | String   |
| `description` | Description of the document | No            | String   |

A consent document self-describing JSON looks like this:

```javascript
{
  schema: 'iglu:com.snowplowanalytics.snowplow/consent_document/jsonschema/1-0-0',
  data: {
    id: 1234,
    version: 5,
    name: 'consent_document_name',
    description: 'here is a description'
  }
}
```

As an example, `trackConsentGranted` will store one consent document as a custom context:

```javascript
window.snowplow('trackConsentGranted',
  1234,                          // Id
  5,                             // Version
  'consent_document',            // Name
  'a document granting consent', // Description
  '2020-11-21T08:00:00.000Z'     // Expiry
);
```

The method call will generate this event:

```javascript
{
  e: 'ue',
  ue_pr: {
    schema: 'iglu:com.snowplowanalytics.snowplow/unstruct_event/jsonschema/1-0-0',
    data: {
      schema: 'iglu:com.snowplowanalytics.snowplow/consent_granted/jsonschema/1-0-0',
      data: {
        expiry: '2020-11-21T08:00:00.000Z'
      }
    }
  },
  co: {
    schema: 'iglu:com.snowplowanalytics.snowplow/contexts/jsonschema/1-0-0',
    data: {
      schema: 'iglu:com.snowplowanalytics.snowplow/consent_document/jsonschema/1-0-0',
      data: {
        id: 1234,
        version: 5,
        name: 'consent_document',
        description: 'a document granting consent'
      }
    }
  }
}
```

<a name="custom-contexts" />

### 3.16 Custom contexts

Custom contexts can be used to augment any standard Snowplow event type, including unstructured events, with additional data.

Custom contexts can be added as an extra argument to any of Snowplow's `track..()` methods and to `addItem` and `addTrans`.

Each custom context is a self-describing JSON following the same pattern as an [unstructured event](#trackUnstructEvent). As with unstructured events, if you want to create your own custom context, you must create a [JSON schema][json-schema] for it and upload it to an [Iglu repository][iglu-repo]. Since more than one (of either different or the same type) can be attached to an event, the `context` argument (if it is provided at all) should be a non-empty array of self-describing JSONs.

**Important:** Even if only one custom context is being attached to an event, it still needs to be wrapped in an array.

Here are two example custom context JSONs. One describes a page, and the other describes a user on that page.

```javascript
{
    schema: "iglu:com.example_company/page/jsonschema/1-2-1",
    data: {
        pageType: 'test',
        lastUpdated: new Date(2014,1,26)
    }
}
```

```javascript
{
    schema: "iglu:com.example_company/user/jsonschema/2-0-0",
    data: {
      userType: 'tester'
    }
}
```

How to track a page view with both these contexts attached:

```javascript
window.snowplow_name_here('trackPageView', null, [{
    schema: "iglu:com.example_company/page/jsonschema/1-2-1",
    data: {
        pageType: 'test',
        lastUpdated: new Date(2014,1,26)
    }
},
{
    schema: "iglu:com.example_company/user/jsonschema/2-0-0",
    data: {
      userType: 'tester'
    }
}]);
```

In this case an empty string has been provided to the optional `customTitle` argument in order to reach the `context` argument.

For more information on custom contexts, see [this][contexts] blog post.

<a name="error-tracking" />

#### 3.17 Error tracking

*Note that this is only available on version 2.7.0 and above of the JS tracker.*

Snowplow JS tracker provides two ways of tracking exceptions: manual tracking of handled exceptions using `trackError` and automatic tracking of unhandled exceptions using `enableErrorTracking`.

<a name="trackError" />

##### 3.17.1 `trackError`

Use the `trackError` method to track handled exceptions (application errors) in your JS code. This is its signature:

```javascript
function (message, filename, lineno, colno, error, contexts)
```

|   **Name** | **Required?** | **Description**                     | **Type**   |
|-----------:|:--------------|:------------------------------------|:-----------|
|  `message` | Yes           | Error message                       | string     |
| `filename` | No            | Filename or URL                     | string     |
|   `lineno` | No            | Line number of problem code chunk   | number     |
|    `colno` | No            | Column number of problem code chunk | number     |
|    `error` | No            | JS `ErrorEvent`                     | ErrorEvent |

Of these arguments, only `message` is required.
Signature of this method defined to match `window.onerror` callback in modern browsers.

```javascript
try {
  var user = getUser()
} catch(e) {
  snowplow_name_here('trackError', 'Cannot get user object', 'shop.js', null, null, e);
}
```

`trackError` can also be passed an array of custom contexts as an additional final parameter. See [Contexts](#custom-contexts) for more information.

Using `trackError` it's assumed that developer knows where error could happen, which is not often the case. Therefor it's recommended to use `enableErrorTracking` as it allows you to discover errors that weren't expected.

<a name="enableErrorTracking" />

##### 3.17.2 `enableErrorTracking`

Use the `enableErrorTracking` method to track unhandled exceptions (application errors) in your JS code. This is its signature:

```javascript
function (filter, contextAdder)
```

|       **Name** | **Required?** | **Description**                         | **Type**                                         |
|---------------:|:--------------|:----------------------------------------|:-------------------------------------------------|
|       `filter` | No            | Predicate function to filter exceptions | Function ErrorEvent => Boolean                   |
| `contextAdder` | No            | Function to grab custom contexts        | Function ErrorEvent => Array<SelfDescribingJson> |

Unlike `trackError` you need enable error tracking only once:

```javascript
snowplow_name_here('enableErrorTracking')
```

Application error events are implemented as Snowplow unstructured events. [Here][application_error] is the schema for a `application_error` event.

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

<a name="trueTimestamps" />

### 3.18 Setting the true timestamp

As standard, every event tracked by the Javascript tracker will be recorded with two timestamps:

1. A `device_created_tstamp` - set when the event occurred
2. A `device_sent_tstamp` - set when the event was sent by the tracker to the collector

These are combined downstream in the Snowplow pipeline (with the `collector_tstamp`) to calculate the `derived_tstamp`, which is our best estimate of when the event actually occurred.

In certain circumstances you might want to set the timestamp yourself e.g. if the JS tracker is being used to process historical event data, rather than tracking the events live. In this case you can set the `true_timestamp` for the event. When set, this will be used as the value in the `derived_tstamp` rather than a combination of the `device_created_tstamp`, `device_sent_tstamp` and `collector_tstamp`.

To set the true timestamp add an extra argument to your track method:

```js
{type: 'ttm', value: unixtimestamp}
```

e.g. to set a true timestamp with a page view event:

```javascript
window.snowplow_name_here('trackPageView', null, [{
    schema: "iglu:com.example_company/page/jsonschema/1-2-1",
    data: {
        pageType: 'test',
        lastUpdated: new Date(2014,1,26)
    }
},
{
    schema: "iglu:com.example_company/user/jsonschema/2-0-0",
    data: {
      userType: 'tester'
    }
}],
{type: 'ttm', value: 1482511723});
```

e.g. to set a true timestamp for a self-describing event:

```javascript
window.snowplow_name_here('trackSelfDescribingEvent', {
    schema: 'iglu:com.acme_company/viewed_product/jsonschema/2-0-0',
    data: {
        productId: 'ASO01043',
        category: 'Dresses',
        brand: 'ACME',
        returning: true,
        price: 49.95,
        sizes: ['xs', 's', 'l', 'xl', 'xxl'],
        availableSince: new Date(2013,3,7)
    }
}, null, {type: 'ttm', value: 1482511723});
```

[Back to top](#top)
[Back to JavaScript technical documentation contents][contents]

[contents]: Javascript-Tracker
[specific-events-v1]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v1
[specific-events-v2.0]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.0
[specific-events-v2.2]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.2
[specific-events-v2.3]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.3
[specific-events-v2.4]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.4
[specific-events-v2.5]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.5
[specific-events-v2.6]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.6
[specific-events-v2.7]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.7
[specific-events-v2.8]: https://github.com/snowplow/snowplow/wiki/2-Specific-event-tracking-with-the-Javascript-tracker-v2.8
[multiple-trackers]: https://github.com/snowplow/snowplow/wiki/1-General-parameters-for-the-Javascript-tracker#24-managing-multiple-trackers
[json-schema]: http://json-schema.org/
[self-describing-jsons]: http://snowplowanalytics.com/blog/2014/05/15/introducing-self-describing-jsons/
[ad-impression-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/ad_impression/jsonschema/1-0-0
[ad-click-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/ad_click/jsonschema/1-0-0
[ad-conversion-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/ad_conversion/jsonschema/1-0-0
[link-click-schema]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/link_click/jsonschema/1-0-1

[openx]: http://www.openx.com/publisher/enterprise-ad-server
[zoneappend]: /snowplow/snowplow/wiki/setup-guide/images/03a_zone_prepend_openx.png
[magicmacros]: http://www.openx.com/docs/whitepapers/magic-macros
[dmp]: http://www.adopsinsider.com/online-ad-measurement-tracking/data-management-platforms/what-are-data-management-platforms/
[contactus]: mailto:contact@snowplowanalytics.com
[gahelppage]: https://support.google.com/analytics/answer/1033863
[gaurlbuilder]: https://support.google.com/analytics/answer/1033867?hl=en
[mixpanel]: https://mixpanel.com/
[kissmetrics]: https://www.kissmetrics.com/
[keen.io]: https://keen.io/
[contexts]: http://snowplowanalytics.com/blog/2014/01/27/snowplow-custom-contexts-guide/
[iglu-repo]: https://github.com/snowplow/iglu

[change_form]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/change_form/jsonschema/1-0-0
[submit_form]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/submit_form/jsonschema/1-0-0
[social_interaction]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/social_interaction/jsonschema/1-0-0
[add_to_cart]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/add_to_cart/jsonschema/1-0-0
[remove_from_cart]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/remove_from_cart/jsonschema/1-0-0
[site_search]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/site_search/jsonschema/1-0-0
[timing]: https://github.com/snowplow/iglu-central/blob/master/schemas/com.snowplowanalytics.snowplow/timing/jsonschema/1-0-0
[performancetiming]: https://github.com/snowplow/iglu-central/blob/master/schemas/org.w3/PerformanceTiming/jsonschema/1-0-0
[performance-spec]: http://www.w3.org/TR/2012/REC-navigation-timing-20121217/#sec-window.performance-attribute
[application_error]: https://raw.githubusercontent.com/snowplow/iglu-central/master/schemas/com.snowplowanalytics.snowplow/application_error/jsonschema/1-0-1