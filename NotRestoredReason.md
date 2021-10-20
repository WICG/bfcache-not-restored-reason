# **NotRestoredReason API Explainer Draft**


## Authors:



*   yuzus@chromium.org
*   bfcache-dev@chromium.org


## Participate



*   [Discussion](https://github.com/whatwg/html/issues/7094)


## Motivation

Browsers today offer an optimization feature for history navigation, called [back/forward cache](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#back-forward-cache) (BFCache). This enables instant loading experience when users go back to a page they already visited. 

Today pages can be blocked from BFCache for different reasons, such as reasons required by spec and reasons specific to the browser implementation. 

Developers can gather the hit-rate of BFCache on their site by using [the pageshow handler persisted parameter](https://html.spec.whatwg.org/multipage/browsing-the-web.html#dom-pagetransitionevent-persisted-dev) and [PerformanceNavigationTiming.type(back-forward)](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigation/type). However, there is no way for developers to tell what reasons are blocking their pages from BFCache in the wild. They are not able to know what actions to take to improve the hit-rate.

We would like to make it possible for sites to collect information on why BFCache is not used on a history navigation, so that they can take actions on each reason and make their page BFCache compatible.


## Developers requirements

The goal is to equip developers with enough information to make their site bfcache compatible.

In order to debug the site, developers need to be able to identify what frame within the frame-tree information applies to. This means they need to be given a tree-structure and ids that match the frame tree. The URL for each frame is helpful for knowing the state of the frame (but cannot be given for a cross-origin iframe).

They need to know whether the frame blocked BFCache or not, and if so the reasons that blocked BFCache.


## Exposing Not-restored reasons in Tree structure

We can report the not-restored reasons in a tree structure representing the frame tree.

For same-origin frames, this can report



1. HTML ID of the frame(e.g. “foo” when&lt;iframe id = “foo” src=”...(URL)”>)
2. URL of the frame
3. Whether or not the frame blocked BFCache
4. Reasons blocking BFCache (can be empty)
5. Child frames

For cross-origin frames, this can report



1. HTML ID of the frame(e.g. “foo” when&lt;iframe id = “foo” src=”...(URL)”>)
2. "src" of the frame (not the current URL)
3. Whether or not the frame blocked BFCache

For cross-origin frames, we should not expose the information on what blocked BFCache to avoid cross-site information leaks. Even when blocked == True, we should not report any reasons.


### **Example-1**


![image](https://user-images.githubusercontent.com/4560413/138027196-d295f3b7-c5fe-4d12-85f3-0b1b91490921.png)



```
{
  URL:"a.com",
  Id: "x",
  blocked: False,
  reasons:[],
  children: [
  	{URL:"a.com", id: "y", blocked: False, reasons:[], children: []},
  	{URL:"a.com", id: "z", blocked: True, reasons:["Broadcast channel"], children: []}
  ]
}
```



### **Example-2 (cross-origin iframes)**


![image](https://user-images.githubusercontent.com/4560413/138027215-b7d17251-732a-457a-8606-8f5ba5dbbf57.png)


```
{
  URL:"a.com",
  Id: "x",
  blocked: False,
  reasons:[],
  children: [
  	{URL:"a.com", id: "y", blocked: False, reasons:[], children: []},
  	{URL:"b.com", id: "z", blocked: True, reasons:[], children: []}
  ]
}
```



## Detailed design discussion


### **Cross-origin iframes**

We don’t want to leak cross-origin information. While exposing things that the outer page knows, i.e. id=”” and src=”” attribute values (reference: [Measure Memory API](https://wicg.github.io/performance-measure-memory/#dictdef-memoryattributioncontainer)), we certainly don’t want to expose the blocking reasons, and it is not clear if we can expose the fact that it blocked BFCache (or not).

But if we expose the same-origin information, that could indirectly imply the BFCache eligibility of the cross-site iframes, so maybe this is fine.


### **Only report blocking frames?**

We could report only the blocking frames (and their parents), instead of reporting the whole tree every time including the non-blocking frames.


### **Specced reasons vs browser specific reasons**

We can report reasons in strings. But we need to make sure that we differentiate between spec-mandated blocking reasons vs browser specific reasons. 

We could split the reasons into two.


```
{"spec reasons": [A, B], "non-spec reasons": [C]}
```



## How to expose data

There are several options on how to expose this data. We could expose via multiple APIs if it makes sense.


### **Pageshow API**

Pageshow API is called every time a page is loaded, and reports the ‘persisted’ parameter to suggest whether it was the initial load or the cache load.

We can extend the pageshow API by reporting the not-restored reasons when persisted == false (BFCache is not used). 


```
window.addEventListener('pageshow', function(event) {
	if (!event.persisted) {
		console.log("BFCache was not used.");
	const reasons = event.notRestoredReasons;
    // [{URL:"a.com", Id: "x", blocked: True, reasons:["broadcast channel"], children:[]}];
}
})
```



### **Reporting API**

[Reporting API](https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API) lets you observe a deprecated feature usage / browser request intervention / crashes.  We can have another category “bfcache” here.


```
let options = {
  types: ['bfcache'],
  buffered: true
}

let observer = new ReportingObserver(function(reports, observer) {
  reportBtn.onclick = () => displayReports(reports);
}, options);
observer.observe();
// -> [{URL:"a.com", Id: "x", blocked: True, reasons:["broadcast channel"], children:[]}]
```



### **Performance Navigation Timing API**

[Performance Navigation Timing API ](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming)tells you the type of navigation (BFCache, prerender). We could also extend this API to report the not-restored reasons.


```
var perfEntries = performance.getEntriesByType("navigation");
for (var i=0; i < perfEntries.length; i++) {
	console.log("= Navigation entry[" + i + "]");
	var p = perfEntries[i];
	// p.notRestoredReason == [{URL:"a.com", Id: "x", blocked: True, reasons:["broadcast channel"], children:[]}]
}
```



## References



*   [Back/Forward Cache web exposed behavior](https://docs.google.com/document/d/1JtDCN9A_1UBlDuwkjn1HWxdhQ1H2un9K4kyPLgBqJUc/edit#heading=h.58d6ijfz2say)
*   [Pageshow API spec](https://html.spec.whatwg.org/multipage/browsing-the-web.html#dom-pagetransitionevent-persisted-dev)
*   [Reporting API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Reporting_API) 
*   [Performance Timing API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming)
*   [Measure Memory API](https://wicg.github.io/performance-measure-memory/#dictdef-memoryattributioncontainer)
