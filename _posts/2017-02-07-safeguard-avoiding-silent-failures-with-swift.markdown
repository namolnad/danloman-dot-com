---
layout: post
title:  "Safeguard: Avoiding 🙉 Silent Failures with Swift"
date:   2017-02-07
categories: swift ios logging
---
Instacart ❤️’s Swift.

For the past year and a half or so, we have been actively moving our customer-facing* iOS codebase from Objective-C to Swift and have all been thoroughly enjoying the transition. There are a lot of advantages gained with the move to Swift and the Swift-ier our code gets, the happier you’ll tend to see us (well, except for Min, he’s our Prince of Darkness… 😈😉)

As with anything, there are a couple of quirks to the language and the evolving conventions that come along with it. Swift, more than almost anything else, is focused on safety. In most cases, as a Swift developer, you want to do whatever it takes to avoid a 💥CRASH💥. The `guard` statement is a great tool for this. If you want to make sure a condition exists prior to executing some code, consider utilizing a `guard` statement, and you’ll likely be safe to assume things are as you’d expect them to be. Crash successfully avoided. Yay!

Using a `guard let` even allows you to unwrap Swift’s Optionals and access their unwrapped values below the `guard`. Awesome. Seriously. We love it.

With `guard let`, however, you find yourself creating a fair amount of these:

{% highlight swift %}
guard let blah = blah else {
    return
}
{% endhighlight %}

This `else` block is a great place to handle the unexpected case, but, when `guard` is used extensively throughout your app, that’s a lot of special cases when you were probably just assuming that you were unwrapping an Optional that should have *some* value. It seems likely you don’t expect that `else` block to run fairly often. So, as is seeming to be the general Swift convention these days, you probably leave it to `else { return }`, except for a few rare cases.

That. That right there. That’s where we see an issue. Unless you are consistently very good about logging or handling the else block appropriately, you basically have no idea how often it’s failing in the wild. 🙀 Ohhh jeez. No one likes being left in the dark like that.

To handle this situation, and give ourselves a general idea of how to track down where issues may be occurring more often than we’d think, we wrote a lightweight framework which we have called **Safeguard.**

The meat and potatoes of what **Safeguard** consists of, is really just a simple extension on Optional with built in logging capabilities that can easily plug into your existing logging system. By implementing the framework's `safeguard()` function in mission-critical areas, we are able to pass important information to our logger to help us track down what the issue may be. `safeguard()` passes the `#function` where it was called, the `#file`, the `#line` and the `Type`. In addition to these clues, with **Safeguard**’s extra `customLoggingParams`, we can pass up relevant session info which may help us reproduce and diagnose the problem later on.

**Safeguard** also provides us with a customizable callback: `nilHandler`, which makes it easy to manage custom use cases when an Optional has failed to unwrap. This `nilHandler` also conveniently passes a `Bool` flag to indicate whether or not the app is running in DEBUG mode. Agh! So many helpful things!!!

To implement `safeguard()`, and give ourselves this nice bit of reassurance, we can add **_all_** of the above functionality to any Optional simply by adding:

{% highlight swift %}
guard let blah = blah.safeguard() else {
    return
}
{% endhighlight %}

Pretty easy. One word added. World saved. Woot woot! 🎉

In the spirit of saving the world and sharing all the goodness, we have [open-sourced][safeguard-repo] **Safeguard**, and have made it easy to install through either Carthage or [Cocoapods][safeguard-cocoapods]. We’ve also included some basic installation, usage and configuration instructions in the [README][safeguard-repo] file. (Of course, if you have any suggestions for improvements or additions, please feel free to add an issue or submit a pull request!)

We hope you enjoy turning all those stupid 🙉 silent failures into 📣 loud failures!

Till next time,

-Dan & the Customer iOS team

```
If you love Swift and are interested in finding solutions like the above, we
are currently hiring mobile engineers and would love to speak with you.

*Note: Our Shopper App team has been working on the same Obj-C -> Swift
transition, credit where it’s due and all that, but I’m on the
customer-facing team and am the one writing this, sooooo, I’ll be writing
from our perspective 😉
```

[safeguard-repo]: https://github.com/namolnad/safeguard
[safeguard-cocoapods]: https://github.com/namolnad/safeguard