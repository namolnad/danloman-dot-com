---
layout: post
title: Use a Gatekeeper to manage control flow
date: 2021-02-13 21:12 -0700
---
A control-flow technique which many of you are likely familiar with goes something like this:

``` swift
class MyViewController: UIViewController {
    private var isLoading: Bool = false

    func load() {
        guard !isLoading else { return }

        isLoading = true

        model
            .fetch()
            .handleEvents(receiveCompletion: { [weak self] _ in
                self?.isLoading = false
            }
        ...
    }
}
```

This methodology—one I've used many times—is plenty effective, but has always felt a bit cumbersome and potentially error prone as there are several areas where you need to toggle the `isLoading` boolean and thusly there are several opportunities to forget to do so. I've long been considering a `propertyWrapper` to encapsulate and streamline this logic, but I struggled to come up with naming and ergonomics which felt like an improvement over the existing solution.

After a bit of brainstorming, I initially wanted to relate this concept to a mutex. However with the help of a colleague, I decided that overloading the mutex name wasn't the right decision. Following some further ideation, I came upon the `Gatekeeper` concept. Essentially, we can imagine there is a human gatekeeper who only allows passage if you haven't already entered, and with whom we must check if we are able to "pass". After toying around with this a bit, I decided there wasn't a need for a propertyWrapper and that a simple struct should suffice.

Here is a simple example of such a type in action:

``` swift
class MyViewController: UIViewController {
    private var loadingGatekeeper = Gatekeeper()

    func load() {
        guard loadingGatekeeper.attemptPassage() else { return }

        model
            .fetch()
            .handleEvents(receiveCompletion: { [weak self] _ in
                self?.loadingGatekeeper.reset()
            })
        ...
    }
}
```

We've now reduced the need for an extra toggling of the `isLoading` boolean as this has been handled by the `Gatekeeper` type during the `attemptPassage()` call. One less chance for errors. 🎉

Of course this is not without its tradeoffs. In this case there may be a slight bit of cognitive overhead understanding the `Gatekeeper` type as well as the intention behind this particular instance. However, I feel the type is simple enough where a developer should be able to quickly understand it in its entirety—and with proper naming of the `gatekeeper` property, the purpose of this instance should remain very straightforward.

As we dive into the declaration of the struct behind this, we can see it is incredibly simple, yet extensible enough to allow for managing various portions of "stateful" control flow.

``` swift
public struct Gatekeeper<Value> {
    private var value: Value
}

extension Gatekeeper where Value == Bool {
    var mayPass: Bool { value }

    init() {
        self.value = true
    }

    @discardableResult
    public mutating func attemptPassage() -> Bool {
        defer { value = false }
        return value
    }

    public mutating func reset() {
        self.value = true
    }
}
```

The boolean extension alone covers off on the majority of simple cases—like the example above—however we can easily add another extension to enable this to work for a much more complex bit of state management.

``` swift
extension Gatekeeper where Value: SetAlgebra {
    public init() {
        self.value = []
    }

    @discardableResult
    public mutating func attemptPassage(for option: Value.Element) -> Bool {
        defer { value.insert(option) }
        return !value.contains(option)
    }

    public func mayPass(for option: Value.Element) -> Bool {
        !value.contains(option)
    }

    public mutating func reset(for option: Value.Element) {
        self.value.remove(option)
    }
}
```

Putting this to use, if we imagine a `State` OptionSet we can see where using a Gatekeeper here may really shine:

``` swift
struct State: OptionSet {
    var rawValue: Int

    static let loading: Self = .init(rawValue: 1 << 0)
    static let loaded: Self = .init(rawValue: 1 << 1)
}

class MyViewController: UIViewController {
    private var stateGatekeeper = Gatekeeper<State>()

    func load() {
        guard stateGatekeeper.attemptPassage(for: .loading), stateGatekeeper.mayPass(for: .loaded) else { return }

        model
            .fetch()
            .handleEvents(receiveCompletion: { [weak self] _ in
                self?.stateGatekeeper.reset(for: .loading)
                self?.stateGatekeeper.attemptPassage(for: .loaded)
            })
        ...
    }
}
```

This will prevent further loads if the `stateGatekeeper` indicates that it is either currently loading or has already been loaded. Pretty cool!

Alright, I'll wrap it up here and leave it to you to see how else you can leverage this simple `Gatekeeper` type. If you find this idea useful or have thoughts on improving the approach, please reach out to me on [Twitter](https://twitter.com/namolnad). I hope that this inspires you to clean up your codebase just a little bit.

Happy coding!
