Types of test pros and cons:
- Unit: fast and low repetition, mocks can couple you to the implementation, and can end up only testing spelling
- Integration: closer to actual usage, but slower and can break
- Acceptance: testing the whole system, what user sees; lot of repetition and slow, can be more brittle b/c UI dependent

Question: how much do you write acceptance vs integration vs unit tests, and in what order?

[BDD:](http://dannorth.net/introducing-bdd/) acceptance test everything, then unit test beneath
- Designed to answer "where to start, what to test and what not to test" (North)
- Focus on analysis/design, so outside in (Fowler)
Classical TDD: integration test starting from the middle, then acceptance
Ward Bell: acceptance test main cases, unit test edge cases
DHH: acceptance test

Bonus: bug reported, replicate with a test first (Bell)

[Adventures in Angular, Ward Bell](Adventures in Angular: 026 AiA Testing Tools
https://overcast.fm/+DcFRMRvTE) - 37ish, core moneymaking functions, Lucas, acceptance test critical path, rigorous unit tests; Ward Bell regression main paths
