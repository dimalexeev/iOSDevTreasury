---
title: "Lorem ipsum. We are trying to find out events of the second type."
description: "There are places of your code which shouldn't be reachable during normal execution process. If that place is reached you should stop program execution and signal about programmer's (probably yours) error. How? Call for fatalError(), assertionFailure() or what?"
date: 2018-10-26T00:14:31+03:00
draft: false
# categories: ["Development", "VIM"]
---

## Problem Description

All events which we call "errors" can be separated to two types.

* Events caused by external factors. Like network connection failure.
* Events caused by programmer's mistake. Like reaching a switch operator case which should be unreachable. 

We process events of the first type in regular control flow. For example we react to network failure by showing message to a user and setting app for waiting of network connection recovery.

We are trying to find out events of the second type as early as possible, before code goes to production. One of the approach here is to terminate program execution in a debuggable sate and print message with indication of where in code the event happened.

And that we can make with the following five functions from the Swift Standard Library (as for Swift 4.2). **Which should we prefer?**

```swift
func precondition(_ condition: @autoclosure () -> Bool, _ message: @autoclosure () -> String = default, file: StaticString = #file, line: UInt = #line)
```
```swift
func preconditionFailure(_ message: @autoclosure () -> String = default, file: StaticString = #file, line: UInt = #line) -> Never
```
```swift
func assert(_ condition: @autoclosure () -> Bool, _ message: @autoclosure () -> String = default, file: StaticString = #file, line: UInt = #line)
```
```swift
func assertionFailure(_ message: @autoclosure () -> String = default, file: StaticString = #file, line: UInt = #line)
```
```swift
func fatalError(_ message: @autoclosure () -> String = default, file: StaticString = #file, line: UInt = #line) -> Never
```

## Solution

[Documentation](https://developer.apple.com/documentation/swift/swift_standard_library/debugging_and_reflection) is ok, but leaves many questions unanswered. Let's look at [source code](https://github.com/apple/swift/blob/master/stdlib/public/core/Assert.swift).

We can see right away the following.

1. Every one of these five functions terminates program execution.
2. Termination happens by two ways.
	* Either with printing convenient debug message by call to `_assertionFailure(_:_:file:line:flags:)`.
	* Or without debug message just by call to `Builtin.condfail(error._value)` or `Builtin.int_trap()`.
3. The difference between our five functions is in the conditions when a call to `_assertionFailure(_:_:file:line:flags:)` is happened and when it isn't.
4. Those conditions are evaluated by calling to the following methods.
	* `_isReleaseAssertConfiguration()`
	* `_isDebugAssertConfiguration()`
	* `_isFastAssertConfiguration()`

Two of those functions are [public](https://github.com/apple/swift/blob/cec6d693374b201ce85d54f92cd38484f99ec55d/stdlib/public/core/AssertCommon.swift) so we may try them giving the following commands in Bash.

```bash
$ echo 'print(_isFastAssertConfiguration())' >conf.swift
$ swift sw.wift
false
$ swift -Osize conf.swift
false
$ swift -Ounchecked conf.swift
true
```

```bash
$ echo 'print(_isDebugAssertConfiguration())' >conf.swift
$ swift sw.wift
true
$ swift -O conf.swift
false
$ swift -Osize conf.swift
false
```

`-O`, `-Ounchecked`, `-Osize` build flags are optimization level instructions for compiler.

* Compile without any optimization: `-Onone` (or none flags given)
* Optimize for Speed: `-O`
* Optimize for Size: `-Osize`

You can set these build flags in Xcode through SWIFT_OPTIMIZATION_LEVEL build setting. ([See here.](https://help.apple.com/xcode/mac/10.0/#/itcaec37c2a6))


**Rule out `-Ounchecked`.** There are [discussions](https://forums.swift.org/t/deprecating-ounchecked/6928) about depricating `-Ounchecked` at all. 

**Rule out conditions.** Two functions let us check for conditions. If condition is failed then function behavior is the same as behavior of its respective cousin: `precondition(_:_:file:line:)` becomes `preconditionFailure(_:file:line)`, `assert(_:_:file:line:)` becomes `assertionFailure(_:file:line:)`.

So we narrow down our analysis to the three functions: `preconditionFailure(_:file:line)`, `assertionFailure(_:file:line)`, `fatalError(_:file:line)`.

---


Why `assertionFailure(file:line)` doesn't return Never?
What returned Never type is for?

---







For example `fatalError(_:file:line)` calls for `_assertionFailure(_:_:file:line:flags:)` unconditionally.

```swift
@_transparent
public func fatalError(
  _ message: @autoclosure () -> String = String(),
  file: StaticString = #file, line: UInt = #line
) -> Never {
  _assertionFailure("Fatal error", message(), file: file, line: line,
    flags: _fatalErrorFlags())
}
```
But `preconditionFailure(_:file:line)` checks for configuration through internal functions `_isDebugAssertConfiguration()` and `_isReleaseAssertConfiguration()`.


```swift
@_transparent
public func preconditionFailure(
  _ message: @autoclosure () -> String = String(),
  file: StaticString = #file, line: UInt = #line
) -> Never {
  // Only check in debug and release mode.  In release mode just trap.
  if _isDebugAssertConfiguration() {
    _assertionFailure("Fatal error", message(), file: file, line: line,
      flags: _fatalErrorFlags())
  } else if _isReleaseAssertConfiguration() {
    Builtin.int_trap()
  }
  _conditionallyUnreachable()
}
```


https://github.com/apple/swift/blob/master/stdlib/public/core/Assert.swift

https://github.com/apple/swift/blob/cec6d693374b201ce85d54f92cd38484f99ec55d/stdlib/public/core/AssertCommon.swift