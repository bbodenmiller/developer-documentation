---
category: Integrate
---

# Single-Page Application Tracking

Single-page websites and web applications have become a standard over the last years. Getting the tracking of such websites and apps right is crucial to your success as you need to ensure the measured data is meaningful and correct.

## Embedding the Tracking Code

First you need to embed your JavaScript tracking code into your single-page website or web application as usual. To do this go to “Administration” in the top right in your Matomo instance, click on “Tracking Code” and adjust the tracking code to your needs.

## Tracking a New Page View

The challenge begins when you need to track a new page view. A single-page application is different from a usual website as there is no regular new page load and Matomo cannot detect automatically when a new page is viewed. This means you need to let Matomo know whenever the URL and the page title changes. You can do
this using the methods `setCustomUrl` and `setDocumentTitle` like this:

```javascript
window.addEventListener('hashchange', function() {
        _paq.push(['setCustomUrl', '/' + window.location.hash.substr(1)]);
        _paq.push(['setDocumentTitle', 'My New Title']);
        _paq.push(['trackPageView']);
});
```
### Resetting previously set custom variables

If you have set any [Custom Variables](https://matomo.org/docs/custom-variables/) in scope “page”, you need to make sure to delete these custom variables again as they would be attributed to the new page view as well otherwise:

```javascript
_paq.push(['deleteCustomVariables', 'page']);
_paq.push(['trackPageView']);
```

Note: we recommend you use [Custom Dimensions](https://matomo.org/docs/custom-dimensions/) instead of Custom Variables as they will be deprecated in the future.

### Updating the generation time

Next you need to update the [generation time](https://matomo.org/docs/page-speed/) before tracking a new page view. Otherwise, the initial page generation time will be attributed to all of your subsequent pageviews. If you don’t load new content from the server when the page changes, simply set the value to zero:

```javascript
_paq.push(['setGenerationTimeMs', 0]);
_paq.push(['trackPageView']);
```
In case you load new content from the server, we recommend measuring the time it took to load this content (in milliseconds) and set the needed time:

```javascript
_paq.push(['setGenerationTimeMs', timeItTookToLoadPage]);
_paq.push(['trackPageView']);
```
### Updating the referrer

Depending on whether you want to track the previous page as a referrer for the new page view, you should update the referrer URL by setting it to the previous page URL:

```javascript
_paq.push(['setReferrerUrl', previousPageUrl]);
_paq.push(['trackPageView']);
```

## Making Matomo Aware of New Content

When you show a new page, your single-page DOM might change as well. For example, you might replace parts of your page with new content that you loaded from your server via Ajax. This means you need to instruct Matomo to scan the DOM for new content. We'll now go over various content types (Videos & Audio, Forms, Links and Downloads, Content tracking).

### Video and Audio tracking

If you use the [Media Analytics](https://matomo.org/docs/media-analytics/) feature to track your videos and audios, whenever a new page is displayed you need to call the following method:

```javascript
_paq.push(['MediaAnalytics::scanForMedia', documentOrElement]);
```
When you don’t pass any parameter, it will scan the entire DOM for new media. Alternatively, you can pass an element to scan only a certain area of your website or app for new media.

### Form tracking

If you use the [Form Analytics](https://matomo.org/docs/form-analytics/) feature to measure the performance of your online forms, whenever a new page is displayed you need to call the following method:

```javascript
_paq.push(['FormAnalytics::scanForForms', documentOrElement]);
```

Where `documentOrElement` points either to `document` to re-scan the entire
DOM (the default when no parameter is set) or you can pass an element to
restrict the re-scan to a specific area.

### Link tracking

Supposing that you use the link tracking feature to measure [outlinks](https://matomo.org/faq/new-to-piwik/faq_71/) and [downloads](https://matomo.org/faq/new-to-piwik/faq_47/), Matomo needs to re-scan the entire DOM for newly added links whenever your DOM changes. To make sure Matomo will track such links, call this method:

```javascript
_paq.push(['enableLinkTracking']);
```

### Content tracking

If you use the [Content Tracking](https://matomo.org/docs/content-tracking/) feature, whenever a new page is displayed and some parts of your DOM changes, you need to call this method :

```javascript
_paq.push(['trackContentImpressionsWithinNode', documentOrElement]);
```
Where `documentOrElement` points either to `document` or an element similar to the other methods. Matomo will then scan the page for newly added content blocks.

### Heatmap & Session Recording

To support single-page websites and web applications out of the box, [Heatmap](https://matomo.org/docs/heatmaps/) & [Session Recording](https://matomo.org/docs/session-recording/) will automatically detect a new page view when you call the `trackPageView` method. This applies if you call `trackPageView` several times without an actual page reload. Matomo will after each call of `trackPageView` stop the recording of any activities and re-evaluate based on the new URL whether if it should record activities for the new page or not. 

If you have a single-page website and you use `trackPageView` for any other purposes than an actual page view, it is recommended to disable the default behaviour using this method and let Heatmap & Session Recording explicitly know when there is a new page view by calling the two methods `disableAutoDetectNewPageView` and `setNewPageView`. Learn more in the [JS Tracker API reference for Heatmaps & Session Recording](https://developer.matomo.org/guides/heatmap-session-recording/reference#disableautodetectnewpageview).

## Measuring Single-Page Apps: Complete Example

In this example we show how everything works together assuming you want to track a new page whenever a hash changes:

```javascript
var currentUrl = location.href;
window.addEventListener('hashchange', function() {
    _paq.push(['setReferrerUrl', currentUrl]);
     currentUrl = '/' + window.location.hash.substr(1);
    _paq.push(['setCustomUrl', currentUrl]);
    _paq.push(['setDocumentTitle', 'My New Title']);

    // remove all previously assigned custom variables, requires Piwik 3.0.2
    _paq.push(['deleteCustomVariables', 'page']); 
    _paq.push(['setGenerationTimeMs', 0]);
    _paq.push(['trackPageView']);
    
    // make Matomo aware of newly added content
    var content = document.getElementById('content');
    _paq.push(['MediaAnalytics::scanForMedia', content]);
    _paq.push(['FormAnalytics::scanForForms', content]);
    _paq.push(['trackContentImpressionsWithinNode', content]);
    _paq.push(['enableLinkTracking']);
});
```
## Questions?

If you have any questions or need help, please [get in touch with our support team](https://matomo.org/support/) or for free support: [ask on the forum](https://forum.matomo.org).
