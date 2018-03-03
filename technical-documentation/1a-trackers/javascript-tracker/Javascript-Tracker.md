<a name="top" />

[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [**Trackers**](trackers) » JavaScript Tracker

## Contents

- 1 [Overview](#overview)  
- 2 [General parameters](1-General-parameters-for-the-Javascript-tracker#wiki-general)  
  - 2.1 [Loading Snowplow.js](1-General-parameters-for-the-Javascript-tracker#wiki-loading)
  - 2.2 [Initialising a tracker](1-General-parameters-for-the-Javascript-tracker#wiki-initialisation)
    - 2.2.1 [Setting the application ID](1-General-parameters-for-the-Javascript-tracker#app-id)
    - 2.2.2 [Setting the platform](1-General-parameters-for-the-Javascript-tracker#wiki-platform)
    - 2.2.3 [Configuring the cookie domain](1-General-parameters-for-the-Javascript-tracker#wiki-cookie-domain)
    - 2.2.4 [Configuring the cookie name](1-General-parameters-for-the-Javascript-tracker#wiki-cookie-name)
    - 2.2.5 [Configuring base 64 encoding](1-General-parameters-for-the-Javascript-tracker#wiki-base-64)
    - 2.2.6 [Respecting Do Not Track](1-General-parameters-for-the-Javascript-tracker#wiki-respect-do-not-track)
    - 2.2.7 [User fingerprinting](1-General-parameters-for-the-Javascript-tracker#wiki-user-fingerprint)
    - 2.2.8 [Setting the user fingerprint seed](1-General-parameters-for-the-Javascript-tracker#wiki-user-fingerprint-seed)
    - 2.2.9 [Setting the page unload pause](1-General-parameters-for-the-Javascript-tracker#wiki-page-unload-timer)
    - 2.2.10 [Altering cookies](1-General-parameters-for-the-Javascript-tracker#session-cookie-duration)
  - 2.3 [Other parameters](1-General-parameters-for-the-Javascript-tracker#other-methods)
    - 2.3.1 [Setting the user id using `setUserId`](1-General-parameters-for-the-Javascript-tracker#wiki-user-id)
    - 2.3.2 [Setting a custom URL with `setCustomUrl`](1-General-parameters-for-the-Javascript-tracker#wiki-custom-url)
  - 2.4 [Managing multiple trackers](1-General-parameters-for-the-Javascript-tracker#wiki-multiple-trackers)
  - 2.5 [How the Tracker uses cookies](1-General-parameters-for-the-Javascript-tracker#wiki-cookies)
- 3 [Specific Event Tracking](2-Specific-event-tracking-with-the-Javascript-tracker)
  - 3.1 [Pageviews](2-Specific-event-tracking-with-the-Javascript-tracker#page)  
    - 3.1.1 [`trackPageView`](2-Specific-event-tracking-with-the-Javascript-tracker#trackPageView)  
  - 3.2 [Pagepings](2-Specific-event-tracking-with-the-Javascript-tracker#pagepings)  
    - 3.2.1 [`enableActivityTracking`](2-Specific-event-tracking-with-the-Javascript-tracker#enableActivityTracking)  
  - 3.3 [Ecommerce transaction tracking](2-Specific-event-tracking-with-the-Javascript-tracker#ecommerce)  
    - 3.3.1 [`addTrans`](2-Specific-event-tracking-with-the-Javascript-tracker#addTrans)  
    - 3.3.2 [`addItem`](2-Specific-event-tracking-with-the-Javascript-tracker#addItem)  
    - 3.3.3 [`trackTrans`](2-Specific-event-tracking-with-the-Javascript-tracker#trackTrans)  
    - 3.3.4 [Pulling it all together: an example](2-Specific-event-tracking-with-the-Javascript-tracker#ecomm-example)
  - 3.4 [Social tracking](2-Specific-event-tracking-with-the-Javascript-tracker#social)
    - 3.4.1 [`trackSocialInteraction`](2-Specific-event-tracking-with-the-Javascript-tracker#trackSocial)
  - 3.5 [Campaign tracking](2-Specific-event-tracking-with-the-Javascript-tracker#campaign)  
    - 3.5.1 [Identifying paid sources](2-Specific-event-tracking-with-the-Javascript-tracker#identifying-paid-sources)  
    - 3.5.2 [Anatomy of the query parameter](2-Specific-event-tracking-with-the-Javascript-tracker#anatomy-of-the-query-parameters)
  - 3.6 [Ad tracking methods](2-Specific-event-tracking-with-the-Javascript-tracker#ad-tracking)
    - 3.6.1 [`trackAdImpression`](2-Specific-event-tracking-with-the-Javascript-tracker#adImpression)
    - 3.6.2 [`trackAdClick`](2-Specific-event-tracking-with-the-Javascript-tracker#adClick)
    - 3.6.3 [`trackAdConversion`](2-Specific-event-tracking-with-the-Javascript-tracker#adConversion)
    - 3.6.4 [Example: implementing impression tracking with Snowplow and OpenX](2-Specific-event-tracking-with-the-Javascript-tracker#ad-example)
  - 3.7 [Tracking custom structured events](2-Specific-event-tracking-with-the-Javascript-tracker#custom-structured-events)  
    - 3.7.1 [`trackStructEvent`](2-Specific-event-tracking-with-the-Javascript-tracker#trackStructEvent)
  - 3.8 [Tracking custom self-describing (unstructured) events](2-Specific-event-tracking-with-the-Javascript-tracker#38-tracking-custom-self-describing-unstructured-events)
    - 3.8.1 [`trackSelfDescribingEvent`](2-Specific-event-tracking-with-the-Javascript-tracker#381-trackselfdescribingevent)   
  - 3.9 [Link click tracking](2-Specific-event-tracking-with-the-Javascript-tracker#link-click-tracking)
    - 3.9.1 [`enableLinkClickTracking`](2-Specific-event-tracking-with-the-Javascript-tracker#enableLinkClickTracking)
    - 3.9.2 [`refreshLinkClickTracking`](2-Specific-event-tracking-with-the-Javascript-tracker#refreshLinkClickTracking)
    - 3.9.3 [`trackLinkClick`](2-Specific-event-tracking-with-the-Javascript-tracker#trackLinkClick)
  - 3.10 [Form tracking](2-Specific-event-tracking-with-the-Javascript-tracker#form-tracking)
    - 3.10.1 [`enableFormTracking`](2-Specific-event-tracking-with-the-Javascript-tracker#enableFormTracking)
  - 3.11 [`trackAddToCart` and `trackRemoveFromCart`](2-Specific-event-tracking-with-the-Javascript-tracker#cart)
  - 3.12 [`trackSiteSearch`](2-Specific-event-tracking-with-the-Javascript-tracker#siteSearch)
  - 3.13 [`trackTiming`](2-Specific-event-tracking-with-the-Javascript-tracker#timing)
  - 3.14 [Enhanced Ecommerce tracking](2-Specific-event-tracking-with-the-Javascript-tracker#enhanced-ecommerce)
    - 3.14.1 [`addEnhancedEcommerceActionContext`](2-Specific-event-tracking-with-the-Javascript-tracker#addEnhancedEcommerceActionContext)
    - 3.14.2 [`addEnhancedEcommerceImpressionContext`](2-Specific-event-tracking-with-the-Javascript-tracker#addEnhancedEcommerceImpressionContext)
    - 3.14.3 [`addEnhancedEcommerceProductContext`](2-Specific-event-tracking-with-the-Javascript-tracker#addEnhancedEcommerceProductContext)
    - 3.14.4 [`addEnhancedEcommercePromoContext`](2-Specific-event-tracking-with-the-Javascript-tracker#addEnhancedEcommercePromoContext)
    - 3.14.5 [`trackEnhancedEcommerceAction`](2-Specific-event-tracking-with-the-Javascript-tracker#trackEnhancedEcommerceAction)
  - 3.15 [Consent event tracking methods](2-Specific-event-tracking-with-the-Javascript-tracker#consent)
    - 3.15.1 [`trackConsentGranted`](2-Specific-event-tracking-with-the-Javascript-tracker#trackConsentGranted)
    - 3.15.2 [`trackConsentWithdrawn`](2-Specific-event-tracking-with-the-Javascript-tracker#trackConsentWithdrawn)
    - 3.15.3 [`Consent documents`](2-Specific-event-tracking-with-the-Javascript-tracker#consent-documents)
  - 3.16 [Custom contexts](2-Specific-event-tracking-with-the-Javascript-tracker#custom-contexts)
  - 3.17 [Error tracking](2-Specific-event-tracking-with-the-Javascript-tracker#errors)
    - 3.17.1 [`trackError`](2-Specific-event-tracking-with-the-Javascript-tracker#trackError)
    - 3.17.2 [`enableErrorTracking`](2-Specific-event-tracking-with-the-Javascript-tracker#enableErrorTracking)
  - 3.18 [Setting the true timestamp](#trueTimestamps)
- 4 [Advanced usage of the JavaScript Tracker](3-Advanced-usage-of-the-JavaScript-Tracker)
  - 4.1 [Callbacks](3-Advanced-usage-of-the-JavaScript-Tracker#callbacks)
  - 4.2 [Methods which can be used from inside a callback](3-Advanced-usage-of-the-JavaScript-Tracker#return-methods)
  - 4.3 [Extracting the Google Analytics cookie ID](3-Advanced-usage-of-the-JavaScript-Tracker#ga)
- 5 [Modifying Snowplow JS](Modifying-snowplow-js)
- 6 [For developers](4-For-developers)


<a name="overview" />

## 1. Overview

The [Snowplow JavaScript Tracker](https://github.com/snowplow/snowplow-javascript-tracker/) works in much the same way as JavaScript trackers for other major web analytics solutions including Google Analytics and Omniture. We have tried, as far as possible, to keep the API very close to that used by Google Analytics, so that users who have implemented Google Analytics JavaScript tags have no difficulty also implementing the Snowplow JavaScript tags.

Tracking is done by inserting JavaScript tags onto pages. These tags run functions defined in [tracker.js](https://github.com/snowplow/snowplow-javascript-tracker/blob/master/src/js/tracker.js), that trigger GET requests of the Snowplow pixel. The JavaScript functions append data points to be passed into Snowplow onto the query string for the GET requests. These then get logged by the Snowplow [collector](collectors). For a full list of data points that can be passed into Snowplow in this way, please refer to the [Snowplow tracker protocol](snowplow-tracker-protocol) documentation.

The JavaScript Tracker supports both synchronous and asynchronous tags. We recommend the asynchronous tags in nearly all instances, as these do not slow down page load times.

2. [General parameters](1-General-parameters-for-the-Javascript-tracker#wiki-general)  
3. [Tracking specific events](2-Specific-event-tracking-with-the-Javascript-tracker#wiki-tracking-specific-events)  
4. [Advanced usage](3-Advanced-usage-of-the-JavaScript-Tracker)  
5. [Modifying Snowplow JS](Modifying-snowplow-js)


[Back to top](#top)  