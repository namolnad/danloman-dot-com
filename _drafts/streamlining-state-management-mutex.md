---
layout: post
title: 'Streamlining State Management: Mutex'
---

A state-management technique which many of you are likely familiar with goes something like this:

``` swift
class MyViewController: UIViewController {
    private var isLoading: Bool = false

    func load() {
        guard !isLoading else { return }

        isLoading = true

        model.fetch()
        ...
    }
}
```

This methodology—one I've used many times—is plenty effective, but has always felt a bit cumbersome and potentially error prone as it could be easy to forget to toggle the isLoading boolean. I've long been considering a propertyWrapper to encapsulate and streamline this logic, but I struggled to come up with naming which felt straightforward.

After a bit of brainstorming, I realized this technique relates quite closely to a `mutex`. Effectively, we could consider that we are trying to claim/lock the mutex for `isLoading`, returning true if we were successfully able to acquire the mutex. If we were successful, the function body could continue execution—though if we were unsuccessful (because we are in the midst of a loading operation), the function would return early. After toying around with this a bit, I decided there wasn't a need for a propertyWrapper and that a simple struct should suffice.

Here is a simple example of this type in action:

``` swift
class MyViewController: UIViewController {
    private var isLoading: Mutex = .init()

    func load() {
        guard isLoading.try() else { return }

        model.fetch()
        ...
    }
}
```

The struct behind this is rather simple, yet extensible-enough to allow for managing various portions of state management.

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
    public mutating func `try`() -> Bool {
        defer { value = false }
        return value
    }

    public mutating func reset() {
        self.value = true
    }
}
```

This covers off on the majority of simple cases, however we can easily add another extension to enable this to work for a much more complex bit of state management.

``` swift
extension Mutex where Value: OptionSet {
    public init() {
        self.value = []
    }

    /// Attempts to acquire the mutex for the given element.
    /// Returns a boolean indicating whether the acquisition was successful.
    @discardableResult
    public mutating func `try`(_ option: Value.Element) -> Bool {
        defer { value.insert(option) }
        return !value.contains(option)
    }

    //public func unless(_ option: Value.Element) -> Bool {
    //    !value.contains(option)
    //}

    public mutating func reset(_ option: Value.Element) {
        self.value.remove(option)
    }
}
```