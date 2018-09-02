## The Cost of 98 KB

Let's think about this bundle size. 128 KB is larger than other frontend view libraries; for example, React is 30 KB gzipped. That's a 98 KB difference, so React is 1/4 the size. That's a real savings, and if Ember doesn't provide you any benefits over React then it certainly makes sense to go with React.


If bundle size is the reason you choose React over Ember, you're giving up all of those benefits…to save 98 KB. And don't forget that your app's total code size is a lot larger than React's 30 KB. Say your app is only 500 KB total. Switching from Ember to React only saves you 20% of the size.

So the question is, how hard are you willing to work for 98 KB? How many hours per week of developer salaries are you willing to spend wiring up libraries yourself, rewriting code from scratch that already exists in Ember addons, upgrading dependencies and troubleshooting version incompatibilities, and training new developers on your custom framework consisting of a hand-picked combination of libraries?

For some applications, all that effort to save 98 KB is worth it. There is a measurable increase in business revenue when bundle size is decreased, because competition is so fierce. But I think the number of such applications is a lot fewer than we think.

If nothing else, we should stop thinking that Ember is a behemoth. Those of us who use it are trading away a whole class of expensive problems for 98 KB of code. That seems like a pretty amazing deal to me.
