---
title: "How to experiment with BFCache NotRestoredReasons in Chrome"
maintainer: "yuzus"
created: 10/11/2022
updated: 10/11/2022
---


# How to experiment with BFCache NotRestoredReasons in Chrome
NotRestoredReasons API is a part of PerformanceNavigationTiming API. You can access why a apge is not restored from BFCache like below:
```javascript
performance.getEntriesByType('navigation')[0].notRestoredReasons;
```
This will get you the results like this:
```javascript
{
  url:"a.com",
  src: "a.com",
  id: "x",
  name: "x",
  blocked: false,
  reasons:[],
  children: [
  	{url:"a.com", src: "a.com", id: "y", name: "y", blocked: false, reasons:[], children: []},
  	{url:"", src: "b.com", id: "z", name: "z", blocked: true, reasons:[], children: []}
  ]
}
```
Read more about the API here: [Explainer](https://github.com/rubberyuzu/bfcache-not-retored-reason/blob/main/NotRestoredReason.md)

## Enabling NotRestoredReasons in Chrome
1. Download a version of Chrome with NotRestoredReasons implemented. ([Link for Canary channel](https://www.google.com/chrome/))
2. Launch Chrome with the command line flag --enable-features=BackForwardCacheSendNotRestoredReasons. ([Instructions for different platforms.](https://www.chromium.org/developers/how-tos/run-chromium-with-flags))

## How to access the NotRestoredReasons value
When the flag is on, NotRestoredReasons field is always present.
It is only populated when a page is not restored from BFCache though.

Navigate to a page, go away from the page and go back to it.
(A.com -> B.com -> A.com)
If A.com is not restored from BFCache, you will see the notRestoredReasons field populated. If not, it will be empty.
```javascript
performance.getEntriesByType('navigation')[0].notRestoredReasons;
```
