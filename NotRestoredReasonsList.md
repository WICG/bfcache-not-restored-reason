<!-- This is a list of NotRestoredReasons that are reported through NotRestoredReasons API. -->
##### x-not-primary-main-frame
Navigation happened in a frame that is not the primary main frame.
##### x-bfcache-disabled
Back/forward cache is disabled by flags.
##### x-related-active-contents
The page was opened using `window.open()` and another tab has a reference to it, or the page opened a window.
##### x-http-status-not-ok
The page's http status code is not 2XX.
##### x-not-http-or-https
The page's URL scheme is not HTTP/HTTPS.
##### x-loading
The page did not finish loading before navigating away.
##### x-granted-media-access
The page has granted access to record video or audio.
##### x-domain-not-allowed
The page's domain is not allowed to enter back/forward cache.
##### x-http-method-not-get
The page was loaded via a method that's not GET.
##### x-subframe-navigation
An iframe on the page started navigation that did not complete upon navigating away.
##### x-timeout
The page exceeded the maximum time in back/forward cache and was expired
##### x-cache-limit
The page was evicted from the cache to allow another page to be cached.
##### x-javascript-execution
The page executed JavaScript while it was in back/forward cache.
##### x-renderer-process-killed
The renderer process for the page in back/forward cache was killed.
##### x-renderer-process-crashed
The renderer process for the page in back/forward cache crashed.
##### x-conflicting-browsing-instance
There was a conflicting browsing instance and the page could not enter back/forward cache.
##### x-cache-flushed
The page in back/forward cache was intentionally cleared.
##### x-service-worker-version-activation
A service worker attempted to send the page in back/forward cache a `MessageEvent`.
##### x-entered-bfcache-before-service-worker-host-added
A service worker was activated while the page was in back/forward cache.
##### x-not-most-recent-navigation
The back/forward navigation was not the most recent navigation and thus could not be restored from back/forward cache.


