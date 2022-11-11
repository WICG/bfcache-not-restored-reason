# **NotRestoredReason API Explainer**


## Authors:



*   yuzus@chromium.org
*   bfcache-dev@chromium.org


## Participate



*   [Discussion](https://github.com/whatwg/html/issues/7094)


## Motivation

Browsers today offer an optimization feature for history navigation, called [back/forward cache](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#back-forward-cache) (BFCache). This enables instant loading experience when users go back to a page they already visited. 

Today pages can be blocked from entering BFCache or get evicted while in BFCache for different reasons, such as reasons required by spec and reasons specific to the browser implementation. 
Here is the full list of reasons that can be reported: [spreadsheet](https://docs.google.com/spreadsheets/d/1li0po_ETJAIybpaSX5rW_lUN62upQhY0tH4pR5UPt60/edit#gid=0).

Developers can gather the hit-rate of BFCache on their site by using [the pageshow handler persisted parameter](https://html.spec.whatwg.org/multipage/browsing-the-web.html#dom-pagetransitionevent-persisted-dev) and [PerformanceNavigationTiming.type(back-forward)](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigation/type). However, there is no way for developers to tell what reasons are blocking their pages from being restored from BFCache in the wild. They are not able to know what actions to take to improve the hit-rate.

We would like to make it possible for sites to collect information on why BFCache is not used on a history navigation, so that they can take actions on each reason and make their page BFCache compatible. First we will start exposing this information to PerformanceTiming API.

The reasons reported can be the ones that were present at the timing of navigating away (i.e. the page did not enter BFCache), or the ones that made the page ineligible while the page was in BFCache (i.e. the page was evicted from BFCache).

Note that **we are not going to expose information about cross-origin subframes, except for the information about whether or not they blocked bfcache**.

## Goals

*   Provide a way to gather data as to why a page is not served from BFCache on a history navigation.
*   Provide an easy way to debug a website and make it BFCache compatible.

## Non-goals

*   Provide a way to disable BFCache.
*   Provide insights into cross-origin subframes.


## Developers requirements

The goal is to equip developers with enough information to make their site bfcache compatible.

In order to debug the site, developers need to be able to identify what frame within the frame-tree information applies to. This means they need to be given a tree-structure and Ids that match the frame tree. The URL for each frame is helpful for knowing the state of the frame (but cannot be given for a cross-origin iframe).

They need to know whether the frame had NotRestoredReasons or not, and if so what reasons are present.


## Exposing Not-restored reasons in Tree structure

We should report the not-restored reasons in a tree structure JavaScript Object representing the frame tree.

For same-origin frames, this should report



1. HTML Id of the frame(e.g. “foo” when&lt;iframe id = “foo” src=”...(URL)”>)
2. name attribute of the frame (e.g. “bar” when&lt;iframe name = “bar”>)
3. Location (URL) of the frame
4. "src" of the frame
5. Whether or not the frame had NotRestoredReason
6. NotRestoredReasons (can be empty)
7. Child frames

For cross-origin frames, this should report

1. HTML ID of the frame(e.g. “foo” when&lt;iframe id = “foo” src=”...(URL)”>)
2. name attribute of the frame (e.g. “bar” when&lt;iframe name = “bar”>, report only the original name, not the updated name)
3. "src" of the frame (not the current URL)
4. Whether or not the frame had NotRestoredReasons

For cross-origin frames, we should not expose the information on what blocked BFCache to avoid cross-site information leaks.
Even when blocked == True, we should not report any reasons.

In addition to this, when there are multiple cross-origin frames that block BFCache, we randomly select one of them and report that it blocked BFCache. This is to minimize the risk of leaking cross-origin information. The details can be found below.


## Examples

### **Example-1**

<img src="https://user-images.githubusercontent.com/4560413/138027196-d295f3b7-c5fe-4d12-85f3-0b1b91490921.png" width="300" height="300">



```
{
  url:"a.com",
  src: "a.com",
  id: "x",
  name: "x",
  blocked: false,
  reasons:[],
  children: [
  	{url:"a.com", src: "a.com", id: "y", name: "y", blocked: "false", reasons:[], children: []},
  	{url:"a.com", src: "a.com", id: "z", name: "z", blocked: "true", reasons:["Broadcast channel"], children: []}
  ]
}
```



### **Example-2 (cross-origin iframes)**

<img src="https://user-images.githubusercontent.com/4560413/138027215-b7d17251-732a-457a-8606-8f5ba5dbbf57.png" width="300" height="300">


```
{
  url:"a.com",
  src: "a.com",
  id: "x",
  name: "x",
  blocked: "false",
  reasons:[],
  children: [
  	{url:"a.com", src: "a.com", id: "y", name: "y", blocked: "false", reasons:[], children: []},
  	/* for b.com */ {url:"", src: "b.com", id: "z", name: "z", blocked: "true", reasons:[], children: []} 
  ]
}
```

### **Example-3 (cross-origin subtree)**
If a cross-origin iframe has a subtree under it, we mask the information of subtree, only reporting the id, src, name and whether or not the subtree blocked bfcache.
This is true even when a subtree has same origin subframe in it, like the example below.
<img src="https://user-images.githubusercontent.com/4560413/177083018-71ac35ed-efb2-4b06-8378-d6ca9fdae621.png" width="300" height="300">

```
{
  url:”a.com”, /* a.com */ 
  src:"a.com",
  id: “x”,
  name: "x",
  blocked: "false",
  reasons:[],
  children: [
  	/* b.com and its subtree */ {url:"", src:”b.com”, id: "y", name: "y", blocked: "false", reasons:[], children: []},
  ]
}
```

### **Example-4 (multiple cross-origin iframes)**

<img src="https://user-images.githubusercontent.com/4560413/201040573-15136c53-b7d9-413a-a5ff-3e93ff7c2d7b.png" width="600" height="300">
If multiple cross-origin iframes have blocking reasons, we randomly select one cross-origin iframe and report whether it blocked BFCache or not. For the rest of the frames, we say "masked" for the blocked value.
See [Security and Privacy](
https://github.com/rubberyuzu/bfcache-not-retored-reason/blob/main/NotRestoredReason.md#single-cross-origin-iframe-vs-many-cross-origin-iframes
) section for more details.

```
{
  url:"a.com",
  src: "a.com",
  id: "x",
  name: "x",
  blocked: "false",
  reasons:[],
  children: [
  	{url:"", src: "b.com", id: "b", name: "b", blocked: "masked", reasons:[], children: []},
  	{url:"", src: "c.com", id: "c", name: "c", blocked: "true", reasons:[], children: []},
	{url:"", src: "d.com", id: "d", name: "d", blocked: "masked", reasons:[], children: []}
  ]
}
```

## Security and Privacy

### **Cross-origin iframes**

We don’t want to leak cross-origin information. While exposing things that the outer page knows, i.e. id=”” and src=”” attribute values (reference: [Measure Memory API](https://wicg.github.io/performance-measure-memory/#dictdef-memoryattributioncontainer)), we certainly don’t want to expose the blocking reasons.

In order not to expose any new cross-origin information, when a cross-origin frame exists in the frame tree, this API will only report whether or not the cross-origin subtree blocked BFCache, and its frame attibutes.

As explained in [Example3](https://github.com/rubberyuzu/bfcache-not-retored-reason/blob/main/NotRestoredReason.md#example-3-cross-origin-subtree) in this explainer, when the frame tree contains a cross-origin subtree, we mask the subtree information; we will not show specific reasons that blocked BFCache and only report that this subtree blocked BFCache.

NotRestoredReasons will be part of window.performance, and this is not accessible from cross-origin subframes. This is reported only to the top main frame.

#### **Single cross-origin iframe vs many cross-origin iframes**
When we expose whether or not a cross-origin iframe blocked BFCache, site authors could potentially infer user's state. For example, when a page embeds an iframe of a social media site and if the iframe's blocking status changes based on user's logged-in state, site authors can tell if the user is logged in or not by this information.

We think exposing a single bit about whether or not a cross-origin iframe blocked BFCache is fine though.
This information - whether or not cross-origin subtree blocked BFCache - is not newly exposed. Site authors could discover this by clearing all other BFCache blocking reasons and then removing the cross-origin subtree before navigating and observing whether the page is BFCached or not.
So giving this bit is not giving away new information, and this information can be useful so that site authors can work with the blocking sites' authors to remove the blockage.

However, when there are many cross-origin iframes, this API could give many bits in one go. For example, a page could embed 20 different social media sites and tell which sites the user is logged in, each bit possibly implying the user state.
This was also technically possible to test before this API, but if we give away the information for all the frames, then that would make it significantly easier for site authors to know this information.

In order to avoid this, we propose to only expose a single bit about cross-origin iframes; that is, if there are multiple cross-origin iframes that block BFCache, we randomly select one iframe and report that it blocked BFCache.
For the rest of the iframes, we would say "Masked".
See [Example4](https://github.com/rubberyuzu/bfcache-not-retored-reason/blob/main/NotRestoredReason.md#example-4-multiple-cross-origin-iframes
)

This way we can minimize cross-origin information leak.

### **Extension usage**

If users have extensions installed and they caused BFCache to be blocked, exposing reasons can be tricky.
There are two levels of new information exposure:

① Users have extensions installed and they are active on this page

② **Specific** extensions are active on this page

① is newly exposed, and maybe it’s okay. 
But we definitely don’t want to expose any signals to detect which extensions are installed and active (②).
We could mask all the reasons related to extensions to say “Extensions blocked BFCache”, so that we don’t give any signal for ② (turning ② into ①). 
There are three possible cases of extensions:

a) Extensions executed script / had unload handlers and blocked BFCache

b) Extensions messaged the page and blocked BFCache

c) Extensions modified the page and as a result blocked BFCache

In case of a) and b), we can mask the specific information and just say “Extensions blocked BFCache”.

In case of c), too, we could say “Extensions blocked BFCache”, instead of a new feature that the page started to use. For example, if an extension modified the page to use IndexedDB and that blocked BFCache, we would not report “Indexed DB usage” but only say “Extensions blocked BFCache”.

If exposing ① Extensions' presence is not okay to expose at all, we can mask all a) b) c) as "Internal error". There are non-extension related reasons that could go into this category, so this will not necessarily expose exnesions' presence.

**After talking to privacy team, we have decided to say “Extensions blocked BFCache” for all of the extension related reasons.**


## Detailed design discussion

### **Only report blocking frames?**

We could report only the blocking frames (and their parents), instead of reporting the whole tree every time including the non-blocking frames.


### **Specced reasons vs browser specific reasons**

We should report reasons in strings. But we need to make sure that we differentiate between spec-mandated blocking reasons vs browser specific reasons. 

We could add "x-" to the browser specific reasons to distinguish them.


```
// foo is browser-specific, bar is specced.
["x-foo", "bar"]
```


## How to expose data

There are several options on how to expose this data. The current plan is to expose it in two ways:

1. Reporting API 

2. Performance Navigation Timing API



### **Reporting API**

[Reporting API](https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API) lets you observe a deprecated feature usage / browser request intervention / crashes.  We would like to have another category “bfcache” here.


```
Report-To: {
             "max_age": 10886400,
             "endpoints": [{
               "url": "a.com"
             }]
           }
// -> [{url:"a.com", id: "x", blocked: true, reasons:["broadcast channel"], children:[]}]
```



### **Performance Navigation Timing API**

[Performance Navigation Timing API ](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming)tells you the type of navigation (BFCache, prerender). We could also extend this API to report the not-restored reasons.


```
window.addEventListener("pageshow", (event) => {
  if (!event.persisted) {
    const navEntries = performance.getEntriesByType("navigation");
    for (var i=0; i < navEntries.length; i++) {
	console.log("= Navigation entry[" + i + "]");
	var p = navEntries[i];
	// p.notRestoredReason == [{url:"a.com", id: "x", blocked: true, reasons:["broadcast channel"], children:[]}]
    }
  }
});
```


## Considered alternatives


### **Pageshow API**

Pageshow API is called every time a page is loaded, and reports the ‘persisted’ parameter to suggest whether it was the initial load or the cache load.

We could extend the pageshow API by reporting the not-restored reasons when persisted == false (BFCache is not used). 

But as per WICG discussion, Performance Navigation Timing API was more preferred, and we are not going to implement this as Pageshow API.


```
window.addEventListener('pageshow', function(event) {
	if (!event.persisted) {
		console.log("BFCache was not used.");
	const reasons = event.notRestoredReasons;
    // [{url:"a.com", id: "x", blocked: true, reasons:["broadcast channel"], children:[]}];
}
})
```


# [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

01.  What information might this feature expose to Web sites or other parties,
     and for what purposes is that exposure necessary?

     > Whether or not the website is blocking back/forward cache or not. 
     > (Though this information is already available using pageshow.persisted)

02.  Do features in your specification expose the minimum amount of information
     necessary to enable their intended uses?

     > Yes.

03.  How do the features in your specification deal with personal information,
     personally-identifiable information (PII), or information derived from
     them?

     > N/A

04.  How do the features in your specification deal with sensitive information?

     > N/A

05.  Do the features in your specification introduce new state for an origin
     that persists across browsing sessions?

     > No.

06.  Do the features in your specification expose information about the
     underlying platform to origins?

     > No.

07.  Does this specification allow an origin to send data to the underlying
     platform?

     > No.

08.  Do features in this specification enable access to device sensors?

     > No.

09.  Do features in this specification enable new script execution/loading
     mechanisms?

     > No.

10.  Do features in this specification allow an origin to access other devices?

     > No.

11.  Do features in this specification allow an origin some measure of control over
     a user agent's native UI?

     > No.

12.  What temporary identifiers do the features in this specification create or
     expose to the web?

     > No.

13.  How does this specification distinguish between behavior in first-party and
     third-party contexts?

     > Expose why the page is not restored from back/forward cache fully in details
     > containing the blocking reasons for first-party contexts.
     > Only expose whether the page blocks back/forward cache or not for third-party
     > contexts.

14.  How do the features in this specification work in the context of a browser’s
     Private Browsing or Incognito mode?

     > No difference.

15.  Does this specification have both "Security Considerations" and "Privacy
     Considerations" sections?

     > It does now: [Security and Privacy](https://github.com/rubberyuzu/bfcache-not-retored-reason/blob/main/NotRestoredReason.md#security-and-privacy
)

16.  Do features in your specification enable origins to downgrade default
     security protections?

     > No.

17.  How does your feature handle non-"fully active" documents?

     > N/A

18.  What should this questionnaire have asked?

     > Is okay to **explicity **expose whether or not cross-origin frames have 
     > blocked back/forward cache?


## References



*   [Back/Forward Cache web exposed behavior](https://docs.google.com/document/d/1JtDCN9A_1UBlDuwkjn1HWxdhQ1H2un9K4kyPLgBqJUc/edit#heading=h.58d6ijfz2say)
*   [Pageshow API spec](https://html.spec.whatwg.org/multipage/browsing-the-web.html#dom-pagetransitionevent-persisted-dev)
*   [Reporting API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API) 
*   [Performance Timing API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming)
*   [Measure Memory API](https://wicg.github.io/performance-measure-memory/#dictdef-memoryattributioncontainer)
*   [TPAC WebPerf WG minutes](https://docs.google.com/document/d/1GQpM8IvL4feXQ0oQdCQIPKhZZkMLNTYJQhBUntMxPkI/edit#heading=h.mo0swzgvknmp)
