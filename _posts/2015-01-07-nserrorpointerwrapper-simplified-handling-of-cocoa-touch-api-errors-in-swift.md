---
layout: post
title: NSErrorPointerWrapper&#58; Simplified handling of Cocoa Touch API errors in Swift
permalink: nserrorpointerwrapper-simplified-handling-of-cocoa-touch-api-errors-in-swift
comments: true
---
The Pod source is here: [NSErrorPointerWrapper](https://github.com/mr-v/NSErrorPointerWrapper/). Read further for motivations and short overview.

Dealing with `NSError` in Objective-C is clunky at best. You have to create a variable of `*NSError` type, then pass it by reference to the potentially erroneous method. That variable may or may not be populated as a result of an action. In fact it's recommended to check the result, not the error, to verify whether the action was successful.

<!--more-->

> Important: Success or failure is indicated by the return value of the method. Although Cocoa methods that indirectly return error objects in the Cocoa error domain are guaranteed to return such objects if the method indicates failure by directly returning nil or NO, you should always check that the return value is nil or NO before attempting to do anything with the NSError object.

Standard way of handling error in Cocoa Touch API in Objective-C looks like this:

```objc
NSError *error = nil;
id result = [object methodWithParameter: parameter, error: &error];
if (result) {
  // do something with the result
} else {
  // handle error
}
```

This is quite a lot of boilerplate code to write/read each time. In Objective-C when result is `nil` it fails the test as if you've passed `false`. Swift is very strict about implicit conversions and you need to make sure whether you're getting a `Bool` or some variation of `AnyObject`, and handle it accordingly.

```swift
var error: NSError?
var result: AnyObject? = methodWithParameter(parameter: parameter, error: &error)
if let unwrapped = result {
  // do something with the unwrapped result
} else {
  // handle error
}
```

To reduce the need to write the same code over and over, I've created a wrapper for calls that need to take in  `NSErrorPointer`. `tryWithErrorPointer` function takes in closure with a single `NSErrorPointer` argument. Sample usage:

```swift
tryWithErrorPointer({ (error: NSErrorPointer) -> Void in
    methodWithParameter(parameter: parameter, error: error) })
```

Using closure syntax goodness you can end up with a simplified call that looks like this:

```swift
tryWithErrorPointer { methodWithParameter(parameter: parameter, error: $0) }
```

where `$0` is a shorthand argument name for the `NSErrorPointer`. OK, that pesky `error` variable is gone, but what about handling success and error? There's a simple DSL that allows you to chain handling of both cases. You can omit any of them as you wish ([example from demo app](https://github.com/mr-v/swift-objc.io-issue-10-core-data-network-application/commit/d881a79126957079c5099efc32ddafd9d10427a1#diff-d11b34ffbe4581d0b14763b5b452fae3L26)).

```swift
tryWithErrorPointer { methodWithParameter(parameter: parameter, error: $0) }
	.onSuccess{ result in 
	  // do something
	}
	.onError{ error in
	  // handle error, log, call abort during development stages, etc.
	}
```

Autocompletion tips us about the handlers and you have less code to write by hand.
Often you need to cast the result to the expected type, since you can't do much with `AnyObject`. Casting is just another thing that can go wrong. Here's a version of `tryWithErrorPointer` that in addition takes in type you'd like to downcast to<sup id="fnref:1">
        <a href="#fn:1" rel="footnote">1</a>
	    </sup>. In case casting fails error in `onError` handler has `NSErrorPointerWrapperFailedDowncast` error code.
	    
```swift
tryWithErrorPointer(castResultTo: NSDictionary.self) { NSJSONSerialization.JSONObjectWithData(JSONData, options: nil, error: $0) }
    .onError{ _ in XCTFail() }
    .onSuccess{ XCTAssertNotNil($0); return }
```

`tryWithErrorPointer` wrapper helps to focus only on the code that's specific to current use case and forget the boilerplate. I think it also reads much better than the standard error handling.

<div class="footnotes"><ol>
    <li class="footnote" id="fn:1">
        <p>Note the `; return` - to override implicit return from closure, which messes up handler`s argument type.<a href="#fnref:1" title="return to article"> ↩</a><p>
    </li>
</ol></div>
