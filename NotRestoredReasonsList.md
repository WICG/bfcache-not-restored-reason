<!-- This is a list of NotRestoredReasons that are reported through NotRestoredReasons API. -->
##### masked
The reason is masked either because the cross-origin iframes in the frame tree
have blocking reasons, or because the main frame has blocked with browser
internal reasons (e.g. implementation details).

##### parser-aborted
While loading the page, parsing was aborted and was incomplete.

##### navigation-failure
Loading the page resulted in error.

##### websocket
There were open WebSocket connections upon leaving the page.

##### weblock
There were held WebLocks upon leaving the page.

##### fetch
Fetch was initiated upon leaving the page.
