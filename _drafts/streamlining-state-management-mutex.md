---
layout: post
title: 'Use a Mutex to manage control flow'
---

_As a primer for this post I should mention that the Mutex type I describe here is not related to thread locks/mutexes and I'm not suggesting reusing/abusing any existing types like `NSLock` in a different context. I'm mean only that we can reuse the **concept** of a Mutex for more simply managing control flow._

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

After a bit of brainstorming, I realized this technique relates quite closely to a `mutex`. Effectively, we could consider that we are trying to claim/lock the mutex for `isLoading`, returning true if we were successfully able to acquire the mutex. If we were successful, the function body continues execution—though if we were unsuccessful (because we are in the midst of a loading operation), the function would return early. After toying around with this a bit, I decided there wasn't a need for a propertyWrapper and that a simple struct should suffice.

Here is a simple example of this type in action:

``` swift
class MyViewController: UIViewController {
    private var loadingMutex = Mutex()

    func load() {
        guard loadingMutex.claim() else { return }

        model
            .fetch()
            .handleEvents(receiveCompletion: { [weak self] _ in
                self?.loadingMutex.release()
            })
        ...
    }
}
```

We've now reduced the need for an extra toggling of the `isLoading` boolean as this has been handled by the `Mutex` type during the `claim()` call. One less chance for errors. 🎉

Of course this is not without its tradeoffs. In this case there may be a slight bit of cognitive overhead understanding the `Mutex` type as well as the intention behind this particular instance. However, I feel the type is simple enough where a developer should be able to quickly understand it in its entirety—and with proper naming of the mutex property, the purpose of this instance should remain very straightforward.

As we dive into the declaration of the struct behind this, we can see it is incredibly simple, yet extensible-enough to allow for managing various portions of "stateful" control flow.

``` swift
public struct Mutex<Value> {
    private var value: Value
}

extension Mutex where Value == Bool {
    init() {
        self.value = true
    }

    /// Attempts to acquire the mutex.
    /// Returns a boolean indicating whether the acquisition was successful.
    @discardableResult
    public mutating func `claim`() -> Bool {
        defer { value = false }
        return value
    }

    public mutating func release() {
        self.value = true
    }
}
```

The boolean extension alone covers off on the majority of simple cases, however we can easily add another extension to enable this to work for a much more complex bit of state management.

``` swift
extension Mutex where Value: OptionSet {
    public init() {
        self.value = []
    }

    /// Attempts to acquire the mutex for the given element.
    /// Returns a boolean indicating whether the acquisition was successful.
    @discardableResult
    public mutating func `claim`(for option: Value.Element) -> Bool {
        defer { value.insert(option) }
        return !value.contains(option)
    }

    public mutating func release(for option: Value.Element) {
        self.value.remove(option)
    }
}
```
