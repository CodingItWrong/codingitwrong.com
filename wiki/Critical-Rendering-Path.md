---
title: Critical Rendering Path
---

* HTTP
	* DNS lookup
	* TLS negotiation
	* Content download
* Rendering
	* Parse HTML into DOM
	* Stylesheets block
	* Create render tree

###  How to not block render

* Link tag “media” attribute to mark stylesheets as not needed
* Stylesheets in the head so the browser can progressively render content (how does it know that’s all the styles?)
* JS at end of page asynchronously
