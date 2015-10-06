---
layout: post
title: Migrating to Swift 2.0 - It's The Little Things
permalink: migrating-to-swift-2
comments: true
---

Wtih Swift 2 now official, it's good time to give migration from 1.2 a go. It can be longer-than-expected process - automatic updater can only get you so far. Instead of whining about it, I'll go over some of the smaller features that don't get as much spotlight as others.

## forEach
Standard library has now `forEach` available for `SequenceType`. Own implementations and hacky use of `map` be gone! Combine this with information about types in collections added to the SDK and we can go from somewhat clunky `deselectCells` implementation:

```swift
func deselectCells() {
    for path in tableView.indexPathsForSelectedRows() as? [NSIndexPath] ?? [] {
        tableView.deselectRowAtIndexPath(path, animated: false)
    }
}
```

to an elegant one:

```swift
func deselectCells() {
    tableView.indexPathsForSelectedRows?.forEach {
        tableView.deselectRowAtIndexPath($0, animated: false)
    }
}
```

No longer there's need to cover for `nil` case, when there are no selected cells. Closure's body will be called only if return value is non-nil.

For such small piece of code we get three improvements:

- seamless `nil` handling,
- collection type information - no more casting,
- less typing and one less (explicit) variable to worry about!


## flatMap

In Swift 1.2 to filter out nil values from an array it was needed to combine filter and map:

```swift
let nilsAreHere: [Int?] = [1, nil, 3, 4, nil]
let noNilsAllowed = nilsAreHere.filter { $0 != nil }.map { $0! }
```

With 2.0 it's been reduced to:


```swift
let nilsAreHere: [Int?] = [1, nil, 3, 4, nil]
let noNilsAllowed = nilsAreHere.flatMap { $0 }
```

## Availibility checking

Say goodbye to more or less hacky ways of checking SDK API`s availibility:

- `respondsToSelector:`,
- `UIDevice.currentDevice().systemVersion.compare`,
- `NSProcessInfo().isOperatingSystemAtLeastVersion` (that was actually no good for iOS 7 checks)
- `NSFoundationVersionNumber < NSFoundationVersionNumber_iOS_8_0`.

Compiler now verifies whether code is accessing new APIs without proper availibility guards around it. Following would throw compilation error if deployment target was set to iOS 7.x:

```swift
NSProcessInfo.processInfo().isOperatingSystemAtLeastVersion(NSOperatingSystemVersion(majorVersion: 8, minorVersion: 0, patchVersion: 0))
```

Registering for push notifications with iOS 7 in mind can be implemented like this:

```swift
if #available(iOS 8.0, *) {
    let types: UIUserNotificationType = [.Alert, .Badge, .Sound]
    let settings = UIUserNotificationSettings(forTypes: types, categories: nil)
    app.registerUserNotificationSettings(settings)
    app.registerForRemoteNotifications()
} else {
    let types: UIRemoteNotificationType = [.Alert, .Badge, .Sound]
    app.registerForRemoteNotificationTypes(types)
}
```

Availibility attributes, besides preventing from one class of crashes, can be used to mark method as deprecated. Once again compiler helps to spot the issue - marking any usage of such method with a warning.


```swift
@available(*, deprecated, message="was deprecated because x, use shinyAction method instead")
func action() {
    // implementation
}
```

### NS_OPTION updates

Foundation's `NS_OPTION` representation in Swift changed from `RawOptionSetType` to `OptionSetType`. It impements `ArrayLiteralConvertible` protocol, so options can be nicely passed using array literal, instead of using `|` binary operator of C heritage.

```swift
// 1.2
let types: UIRemoteNotificationType = .Alert | .Badge | .Sound
// 2.0
let types: UIRemoteNotificationType = [.Alert, .Badge, .Sound]
```

While above change is purely stylistic, in the process the need for `.allZeros` option was eliminated. Just pass an empty array if there are no options. Nice small step towards a cleaner API.

```swift
NSLayoutConstraint.constraintsWithVisualFormat(format, options: [], metrics: nil, views: binding)
```

### Honorable Mentions

There are a lot more changes worth mentioning. Won't go over them, just attaching a table showing how nicer Swift's Standard Library and working with Cocoa Touch got.

|1.2 | 2.0|
|:--:|:--:|
|`kCGBlendModeDestinationIn` | `CGBlendMode.DestinationIn` |
|`view.setTranslatesAutoresizingMaskIntoConstraints(false)`|`view.translatesAutoresizingMaskIntoConstraints = false` |
|`find(controllers, viewController)`|`controllers.indexOf(viewController)`
|`“2”.toInt`|`Int(“2”)`|
|`var list = ["4", "3", "2", "1"]; list.sort(<)`|`var list = ["4", "3", "2", "1"]; list.sortInPlace()`|
|`frame.rectByOffsetting(dx: dx, dy: dy)`|`frame.offsetBy(dx: dx, dy: dy)`|
