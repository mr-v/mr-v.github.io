---
layout: post
title: Xcode 7/Swift 2 unit testing setup
permalink: xcode-7-swift-2-unit-testing-setup
comments: true
---

Automated tests save time and give confidence when refactoring code.
How one would go about starting writing tests in Swift? There's a bunch of ways to do unit testing, different styles (TDD, BDD), different matchers. Before delving into any of this, let's make sure our testing setup is up to date with Swift 2/Xcode 7 capabilities.

<!--more-->

## Testing Internal Methods

For the first year of Swift developers were using different workarounds to test internal methods:

- Adding file under test to the test target itself. Ugh, that's one more thing to remember and manage. We don't want to do this, just compile once and use it.
- Adding `public` access modifier to make methods visible to other modules. Well, that changes the original intent altogether, so it's also a no-no.

With Swift 2 this was solved by introducing `@testable` attribute. To be able to use it, app target needs testability enabled, see `App Target -> Build Settings -> Build Options -> Enable Testability`. In projects created with Xcode 7 it's on by default. Double check `Product Module Name` and use it to import app target in test case:

```swift
@testable import AwardWinningApp
```

Now all internal entities of `AwardWinningApp` module are available inside test target. Simple solution, no workarounds needed.


## Speeding Up Tests

When running tests inside of "host application", it's a good practice to stop usual app initialization. We're unit testing here, no need to run the full app with authentication, networking and building up views. One way of fixing this is putting early returns inside of app delegate. Thought that there must be a better way do this. Searching lead me to Jon Reid's ["How to Easily Switch Your App Delegate for Testing"](http://qualitycoding.org/app-delegate-for-tests/) blog post. In there he recommends to switch out app delegate in `main` function. Instead of default delegate, use one provided by the test target. `TestingAppDelegate` is just a stub that does nothing. This way replacement is done at the earliest possible moment, no additional checks or hacks required.

But that's Objective-C, how to do this in Swift? In the comments Paul Booth shared his solution:

> Comment out @UIApplicationMain in AppDelegate.swift

```swift
// TestingAppDelegate.swift
import UIKit

class TestingAppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
}

// main.swift
import Foundation
import UIKit

let isRunningTests = NSClassFromString("XCTestCase") != nil

if isRunningTests {
    UIApplicationMain(C_ARGC, C_ARGV, nil, NSStringFromClass(TestingAppDelegate))
} else {
    UIApplicationMain(C_ARGC, C_ARGV, nil, NSStringFromClass(AppDelegate))
}
```

That works in Xcode 6.4/Swift 1.2. We're getting closer. But whoah, what's this `main.swift`?! _Swift Programming Language_ answers this:

>  UIApplicationMain

> Apply this attribute to a class to indicate that it is the application delegate. Using this attribute is equivalent to calling the UIApplicationMain function and passing this class’s name as the name of the delegate class.

> If you do not use this attribute, supply a main.swift file with a main function that calls the UIApplicationMain(_:_:_:) function. 

In Xcode 7 some things got moved around and test target module is not loaded when `UIApplicationMain` gets called. Using `NSStringFromClass` to instantiate `TestingAppDelegate` is a no go in this case. But do we actually need to provide this stub delegate?  Turns out we can just pass `nil`. Checking for `XCTestCase` class presence is good enough way of veryfing whether tests are running. As of now this doesn't clash with Xcode's UI Testing target: during UI tests `UIApplicationMain` is called when `XCTest` framework is not yet loaded. Though it may change, as seen above.

With those changes and Swift 2 updates, I've arrived at bare minimum implementation for stubbing app delegate for tests:

```swift
import UIKit

private func delegateClassName() -> String? {
    return NSClassFromString("XCTestCase") == nil ? NSStringFromClass(AppDelegate) : nil
}

UIApplicationMain(Process.argc, Process.unsafeArgv, nil, delegateClassName())
```

That's nice.

We barely got accustomed to seeing Xcode 7 without beta ribbon, when Apple came out with Xcode 7.1 and Swift 2.1. In Xcode 7.1 beta 2 release notes hides another gem, that may speed up testing feedback loop:

>  Editing a file no longer triggers a recompile of files that depend on it if the edits only modify declarations marked private. (22239821)

Great to see this improvement in the compiler.