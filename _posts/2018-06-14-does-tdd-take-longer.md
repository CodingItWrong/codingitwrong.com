—-
title: Does Test-Driven Development Take Longer?
—-

Recently a coworker asked me if Test-Driven Development (TDD) takes longer than not doing TDD. I’ve learned that the costs and benefits of testing vary in different environments. But at least in environments where TDD is a good fit, here are my thoughts:

1. When you’re doing TDD *while learning it*, it takes longer than if you weren’t doing TDD. But this is just temporary; once you know TDD, you won’t be slowed down by the learning process.

2. While writing tests takes time, it can take the place of time-consuming manual testing. Spending a minute to write a test that reruns a dozen times during feature development can already pay off if you otherwise would have done a thirty-second manual testing process in the browser a dozen times.

3. The biggest bet of TDD is that it saves you time in the long run by improving the quality of your code. When code is overly complex or poorly structured, working in it is slow: it takes a long time to understand, and making changes tends to cause things to break. Tests help you avoid these problems by influencing your code quality in the first place and by allowing you to refactor safely when code quality problems come up.
