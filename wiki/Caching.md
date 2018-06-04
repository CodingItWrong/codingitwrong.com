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
	* Max-Age
	* Last-Modified https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified
* Cache busting
* Application Cache https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache
	* Deprecated
* Service Workers https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers
	* Better offline support
	* More granular control
	* Preloading
	* Background updating

Google Page Speed Insights

http://designingforperformance.com/optimizing-markup-and-styles/#css-and-javascript-loading

http://alistapart.com/article/application-cache-is-a-douchebag

https://stackoverflow.com/questions/35190699/whats-the-difference-between-using-the-service-worker-cache-api-and-regular-bro#35190700

Service Worker

It's being removed from create-react-app as a default in the upcoming version.

<https://github.com/facebook/create-react-app/issues/2398#issuecomment-358826045>

Here's my summary of some of the problems as reported in that thread

If you aren’t aware, not clear why refreshing/stopping/starting another server doesn’t change things. Especially preventing another local app from being seen.

Root problems
- It’s a fundamental change to the way the browser behaves for users and developers
- It’s caching, and caching is hard
- If you get it wrong, your app can’t be updated at all unless users manually remove the service worker
- Caching static assets is fine, but if data isn’t available you have to code for it.

It’s not an introductory level thing. It’s added complexity that you can add when the benefit outweighs the cost.

Ultimately, offline support makes it a distributed system, and those are inherently complex. The web has been easy because it’s assumed online and the browser handles offline status. Mobile apps are more complex because you need to code for many different scenarios.

“it is not safe to enable this feature unless you can control Cache Headers for service-worker.js - which is out of responsibility of c-r-a”

“That technology, used properly, has definitely huge advantages. Misused, it has huge consequences. I'm gonna stick with the normal/standard/toBeExpected behavior of my websites/apps for now. I'll dive into it and run proper tests and experimentation when I actually have the time&interest to do so.”
