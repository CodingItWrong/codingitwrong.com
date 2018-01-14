---
title: Deferred Loading
---

## Post-Render Loading, hydration

Unblock rendering for optimal user experience. But beware the uncanny valley of non-interactive pages! Gap events

* async attribute
    * Load order not guaranteed, so one script can’t depend on another
    * Good for stand-alone scripts like analytics
* defer attribute
	* Still executed in order
	* Low browser support?
* <https://stackoverflow.com/questions/10808109/script-tag-async-defer#10808243>

## Lazy Loading

Loading features/sections of your JS app as requested.

* Dynamic import()
  * [Proposed to TC39](https://github.com/tc39/proposal-dynamic-import)
  * [Available via Webpack](https://webpack.js.org/guides/code-splitting/)
* [SystemJS](https://github.com/systemjs/systemjs/blob/master/README.md)
* Not broadly supported
  * &lt;script type="module"&gt; - [(not Firefox)](http://caniuse.com/#feat=es6-module)
  * <https://hacks.mozilla.org/2015/08/es6-in-depth-modules/>
