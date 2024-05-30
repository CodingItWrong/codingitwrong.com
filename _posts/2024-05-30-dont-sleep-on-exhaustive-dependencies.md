---
title: "Don't Sleep on Exhaustive Dependencies"
---

In React, if you have a `useEffect` hook that accesses a dependency not listed in the dependencies array, the lint rule [`react-hooks/exhaustive-deps`](https://github.com/facebook/react/blob/main/packages/eslint-plugin-react-hooks/README.md) (assuming you have it enabled) will give the error:

> React Hook useEffect has a missing dependency: '…'. Either include it or remove the dependency array.

Oftentimes, following the suggestion and directly adding the dependency array can lead to the effect being run more frequently than you want, or even an infinite loop. When this happens, I often see developers simply disable the lint rule for this line. When asked why, they often simply say that the code breaks unless the rule is disabled. When I tell them that there should be a way to rearrange the code to satisfy the rule, I often get the response that this makes the code much more complex, and that it isn’t worth the effort to do so.

So should you just disable the exhaustive-deps rule at places where it’s causing you issues? Or maybe disable the rule entirely? I don’t think so. Here’s why.

The rule about exhaustive dependencies (not just the ESLint rule, [but general rule about how to use dependencies](https://react.dev/reference/react/useEffect#specifying-reactive-dependencies)) was put in place by the React core team because it gets at something important about what effects are intended to be, and the situations in which they’re intended to be used. If we use effects in a way that the React core team is explicitly warning us _not_ to use them, we risk bugs in our code now, breakage in the future, and making it harder for other React developers to understand our code.

So what would I recommend instead? First, see if you can avoid using an effect entirely, because often (but not always) there’s a better alternative. Second, if you find you _do_ need to use an effect, think about it from the mental model the React docs recommend, and let that guide how you design it in a way that will satisfy the exhaustive-deps rule.

First, see if you can avoid using an effect. The React docs have an entire page dedicated to this topic: [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect). It’s worth reading in its entirety. A few key cases that often come up (see the page for more explanation and examples):

- **To load data** from an API, use a React framework or a data-fetching library.
- **To calculate data** from other state or props, just perform that calculation directly in the render function instead of in an effect. If you find this causes a performance issue, memoize the data.
- **To respond to an action** the user took, put that logic in the event handler function that is called immediately in response to the action (`onClick`, etc)
- **To reset all the state** when a prop changes (often in the case of a form), set a `key` on the component to force React to create a new instance of the component.

The reason to avoid using an effect is not because they’re bad. There are two reasons: first, they are complex to use—that’s why the React team feels like the exhaustive-deps rule is needed, and that’s why we often feel like satisfying that rule will require a lot of work. Second, effects aren’t really designed for these scenarios. According to the React docs, what effects are for is [synchronizing with external systems](https://react.dev/learn/synchronizing-with-effects)—that’s all. So if you need to do something other than synchronizing with an external system, if there’s a way to do it _other_ than an effect, it’s best to do it that way. And if effects are the only way to do it (as in the case of client-side data loading), it’s best for a library to implement it once, and that way they can put all the effort into handling any edge cases to make sure it’s reliable.

If you can’t avoid using an effect, design your effect in a way that will satisfy the exhaustive-deps rule. I wish I could come up with more helpful instructions on how to do this, but I haven’t been able to so far—it’s just a hard problem. I’ll give some general thoughts here, but I’d also recommend finding your favorite React trainer or instructor and checking out any resources they have on useEffect.

useEffect is designed to synchronize with external systems based on the updated dependencies. So, basically, every time the dependencies change, your cleanup function (if present) runs to “tear down” the previous synchronization, and then the hook runs with the new deps to set up the _new_ synchronization. React wants you to tell it “run this code for any given state of these dependencies.” So if an effect is “running too often,” that means you have some logic that you want to run for _some_ states of dependencies and not others. Well, we already have a way to run code sometimes but not others: the simple `if` statement. You can add conditionals into the effect’s function to check the values of the dependencies and only run a given bit of logic sometimes, not every time a dependency changes. In other words, it's not that "the effect" is running too often, it's that your bit of logic is: and you can use conditionals to get it to run only when you want to. However, these conditionals can get complex; that's part of the pushback.

If you find you need an effect and you just don’t have time right now to design it to satisfy the exhaustive-deps rule, at least leave a `TODO` comment to remind you to do so in the future. It could be a good idea to also record what specifically breaks when you add in the extra dependencies.
