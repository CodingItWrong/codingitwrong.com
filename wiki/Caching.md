---
title: Caching
---

For quicker second load and for preparation for offline support

* Browser cache
	* Controlled by cache-related response headers
	* https://jakearchibald.com/2016/caching-best-practices/
	* Cache-Control https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
	* ETags https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag
	* Expires https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expires
	* Last-Modified https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified
* Cache busting
* Application Cache https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache
	* Deprecated
* Service Workers https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers
	* Better offline support
	* More granular control
	* Preloading
	* Background updating

http://designingforperformance.com/optimizing-markup-and-styles/#css-and-javascript-loading

http://alistapart.com/article/application-cache-is-a-douchebag

https://stackoverflow.com/questions/35190699/whats-the-difference-between-using-the-service-worker-cache-api-and-regular-bro#35190700
