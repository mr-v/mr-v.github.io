---
layout: post
title: Custom Error Assertions in Swift
permalink: custom-error-assertions-in-swift
comments: true
---

Swift 2 introduced revamped error handling <sup id="fnref:1"><a href="#fn:1" rel="footnote">1</a></sup>. Unfortunately XCTest framework has not been updated for this occasion. Without any custom libraries developers are stuck with tests like those:

~~~swift
func test_throwOnUnkown_UnknownPassed_ThrowsAnyError() {
    do {
        try Movie().throwOnUnkown(.Unkown)
        XCTFail()
    } catch {
    }
}

func test_throwOnUnkown_UnknownPassed_ThrowsTestError() {
    do {
        try Movie().throwOnUnkown(.Unkown)
        XCTFail()
    } catch {
        guard let _ = error as? TestError else {
            return XCTFail()
        }
    }
}

func test_throwOnUnkown_UnknownPassed_ThrowsIllegalArgument() {
    do {
        try Movie().throwOnUnkown(.Unkown)
        XCTFail()
    } catch {
        guard let error = error as? TestError else {
            return XCTFail()
        }
        XCTAssertEqual(TestError.IllegalArgument, error)
    }
}
~~~

Example is very rudimentary, however those tests still read badly. Wouldn't [this alternative](https://github.com/mr-v/AssertThrows) be much nicer?

~~~swift
func test_throwOnUnkown_UnknownPassed_ThrowsAnyError() {
    let movie = Movie()
    AssertThrows(try movie.throwOnUnkown(.Unkown))
}

func test_throwOnUnkown_UnknownPassed_ThrowsTestError() {
    let movie = Movie()
    AssertThrows(TestError.self, try movie.throwOnUnkown(.Unkown))
}

func test_throwOnUnkown_UnknownPassed_ThrowsIllegalArgument() {
    let movie = Movie()
    AssertThrows(TestError.IllegalArgument, try movie.throwOnUnkown(.Unkown))
}
~~~

So how to get rid of unwieldy tests from first example and arrive at second solution?

<!--more-->

First some motivations. I've recently been reading [Working Effectively with Unit Tests](https://leanpub.com/wewut) book that tackles similar problem in Java. Its exception handling syntax is quite similar to error handling in Swift, so this and other advice on asserting from the book applies here as well.

First one is to use [Arrange-Act-Assert](http://c2.com/cgi/wiki?ArrangeActAssert) pattern for structuring unit tests. In the book main takeaway is that assertion should always be last part of the test. When reading unit test for the first time, it's a good idea to start with assertion<sup id="fnref:1"><a href="#fn:1" rel="footnote">2</a></sup>. Writing tests can be also started with assertion, to clearly drive test's development. In both cases assertion seems like a good entry point for developer. Following this structure brings consistency to the test suite, making it easier to maintain. We seek assertion at the end and know immediately what is tested.

For contrast let's take excerpt from the first example:

~~~swift
do {
    try Movie().throwOnUnkown(.Unkown)
    XCTFail()
} catch {
}
~~~

Here the test passes if control goes to empty catch block - not very intuitive, there's something off about it.

Another good advice is to have only single assertion per test.

~~~swift
func test_throwOnUnkown_UnknownPassed_ThrowsIllegalArgument() {
    do {
        try Movie().throwOnUnkown(.Unkown)
        XCTFail()
    } catch {
        guard let error = error as? TestError else {
            return XCTFail()
        }
        XCTAssertEqual(TestError.IllegalArgument, error)
    }
}
~~~

Method above includes three (!) assertions - in larger tests much time can be spent on assessing  conditions for test to pass, since there are multiple points of failure. We just want to test a single thing! It's also easy to imagine that one `XCTFail` call may be lost during editing or copying/pasting (tests with cumbersome structure encourage such behaviour). We may end up with false positive test then, at best we will discover this and loose our trust in test suite.

Another issue is that those tests are very verbose and similar in structure. I'd have no desire to read and maintain them. Trying to figure out subtle differences between them would be at best tedious. Let's remember that test case here is a very simple one. Only single throwing function, single `ErrorType` implementer and its single case get verified. 

Here comes another advice from [Working Effectively with Unit Tests](https://leanpub.com/wewut):

> Custom assertion can take _assert_ structural duplication and replace it with concise , globally useful single assertion.

Structural duplication. That's exactly what's going on here. Verifying errors involves a lot of `do-catch`-`XCTFail` dance. Custom assertions are easier to work with: they offer familiar structure and if test fails, there's only a single place to go to.

Following tests make sense for throwing function in Swift:

1. Throws error.
2. Throws specific `ErrorType` implementer.
3. Throws specific case of `ErrorType` implementer.

Ideally we want to test expected behaviour in a single line.

~~~swift
AssertThrows(TestError.IllegalArgument, try movie.throwsOnUnknown(.Unknown))
~~~

Let's once again take a look at verbose version, as it will be starting point for implementing  `AssertThrows` for the third case.

~~~swift
func test_throwOnUnkown_UnknownPassed_ThrowsIllegalArgument() {
    do {
        try Movie().throwOnUnkown(.Unkown)
        XCTFail()
    } catch {
        guard let error = error as? TestError else {
            return XCTFail()
        }
        XCTAssertEqual(TestError.IllegalArgument, error)
    }
}
~~~

Swift provides us with great tools to change this specific piece of code into generalized assertion. It's still similar in structure, but will be hidden behind nice, single line `AssertThrows` call. It can be tested, so if something fails we know custom assertion is rock solid and the issue lies with the new code we've just written. See implementation below.

~~~swift
public func AssertThrows<T where T: Equatable, T: ErrorType>(expected: T, @autoclosure _ throwingCall: () throws -> (), file: String = __FILE__, line: UInt = __LINE__) {
    do {
        try throwingCall()
        XCTFail()
    } catch {
        guard let castedError = error as? T where expected == castedError else {
            return XCTFail("AssertThrows: (\"\(expected.dynamicType) \(expected)\") is not equal to (\"\(error)\")", file: file, line: line)
        }
    }
}
~~~

Just a few lines of code, but they greatly clean up error testing codebase. Sketchy looking logic is buried deep in `AssertThrows` implementation, we no longer need to remember it and watch our step. As mentioned, adding [a few simple tests](https://github.com/mr-v/AssertThrows/tree/master/AssertThrowsTests) of `AssertThrows` provides a safety net to ensure that assertion itself works as advertised.

With just a few changes, we've arrived at general solution - fully working custom error handling assertion. I love this side of working with Swift. There are a lot of Swift language features crammed up into this piece of code:

- generics with [type constraints](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html#//apple_ref/doc/uid/TP40014097-CH26-ID186) requiring protocols conformity,
- [@autoclosure](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-ID543) for nice assertion syntax,
- using default `file` and `line` arguments to display failures on proper lines in Xcode (passing [built-in identifiers](https://developer.apple.com/swift/blog/?id=15)),
- `guard` statement with optional binding and `where` clause.

All of this helped with patching up XCTest's missing parts. Now we can enjoy quality error handling testing!

Source code is avaible on [GitHub](https://github.com/mr-v/AssertThrows).


p.s

There's one caveat - `ErrorType` conforming enumerations with associated data, must also implement `Equatable` protocol to be verified with `AssertThrows`. Standard enums have it for free.


***

<div class="footnotes"><ol>
    <li class="footnote" id="fn:1">
        <p><a href="https://github.com/apple/swift/blob/master/docs/ErrorHandlingRationale.rst">Error Handling Rationale and Proposal</a> makes a good reading. Available due to Swift's open-sourcing!<a href="#fnref:1" title="return to article"> ↩</a><p>
    </li>
    <li class="footnote" id="fn:1">
        <p>Note: Test method name is also a good place, but they are susceptible to the same problem as comments.<a href="#fnref:1" title="return to article"> ↩</a><p>
    </li>
</ol></div>
