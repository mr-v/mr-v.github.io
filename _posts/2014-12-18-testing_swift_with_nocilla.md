---
layout: post
title: HTTP testing in Swift with Nocilla
permalink: http-testing-in-swift-with-nocilla
comments: true
---
In this post you'll learn about:

- testing networking layer with Nocilla and XCTest,
- using Objective-C libraries with Swift,
- new XCTest asynchronous testing API,
- CocoaPods,
- basics of test naming and structure.

<!--more-->

## Testing networking code
There are a few problems with unit testing the networking layer:

- results are not 100% reliable (tests may fail due to a number of factors - bad connection, server downtime),
- test are slow - any external dependencies (networking, disk access, etc.) cause tests to be slower, than when they would run solely in-memory.
It may be also hard to setup and test specific conditions, especially if you want check something other than the "happy path" ("Does this error handling code works like it supposed to?").

Solution to this is to stub the networking to mitigate the need to hit the network, or even having a working server. Nocilla is a library that stubs HTTP requests and responses. It has great API with custom DSL that allows to fluently compose the stubs. It's written in Objective-C, but due to Swift - Objective-C interoperability there's no problem with using it for tests in Swift projects. Easiest way to integrate a third party dependency to a project is with CocoaPods.

## Installation Guide
As usage instructions on [Nocilla's github](https://github.com/luisobo/Nocilla) are written for Kiwi framework and for Objective-C I decided to write a guide on how to set it up in Swift project and use with vanilla XCTest framework.
Steps:

- Install [CocoaPods](http://cocoapods.org/) (if you haven't already)
- In projects top level folder execute command: "pod init"
  - Among other things it will create a Podfile; edit it with your text editor of choice and add Nocilla to your test target. In my case test target is named ServiceTests and Podfile looks like this:

```ruby
target 'ServicesTests' do
pod 'Nocilla'
end
```

- To install CocoaPods dependencies call "pod update". After this projects have to be run from newly created '(project name).xcworkspace'.
- To use Objective-C code in Swift a bridging header is needed. Add imports to this header for any libraries you'd like to be visible in Swift.
  - Simplest way of setting up this header is to add empty Objective-C file (New File -> Source). Xcode will ask you if you want a bridging header. Sure you do! After this you delete just added empty file. This helps to avoid creating header manually (and potentially messing up paths).
  - Add #import "Nocilla.h" to generated header - now it's accesible in Swift.

## Test suite setup and testing asynchronous code
I've created NocillaTestCase deriving from XCTest case, to encapsulate all the setup that's needed to be done when testing with Nocilla. Class methods setUp and tearDown make sure the library is stubbing only during this test suit:

```swift
override class func setUp() {
    super.setUp()
    LSNocilla.sharedInstance().start()
}

override class func tearDown() {
    super.tearDown()
    LSNocilla.sharedInstance().stop()
}
```

Stubs get cleared between tests of the suite.

```swift
override func tearDown() {
    // ...
    LSNocilla.sharedInstance().clearStubs()
}
```

Due to the fact that "NSURLSession API is highly asynchronous" we need to use the new XCTest API and setup an expectaction before each test. Expectaction signals that there's async code to be executed. At the end of the test call `waitForExpectationsWithTimeout`, so the function won't return and execute next test. It waits until expectaction is fullfiled (you need to call it manualy on async code completion) or if it times out. When running tests with Nocilla, timeout can be set to very low values (under 0.1). Even if it fails it's still faster than running a network call - response is almost immediate.
Add it at the end of the test:

```swift
waitForExpectationsWithTimeout(0.1, handler: nil)
```

Passing nil to handler is enough for tests to fail in case of timeout. If you need to do any additional work in handler, be sure to check if hadnlers error parameter is non nil and call XCTFail assertion, as it's invoked both on expectaction fullfilment and timeout.

As each test in a suite will use asynchronous API NocillaTests create new expectaction in setUp instance method and clean it up in the tearDown. Note: to avoid creating initializer <sup id="fnref:1">
        <a href="#fn:1" rel="footnote">1</a>
    </sup> expectaction is declared as an explicitly unwrapped XCTestExpectation.

## Testing HTTP requests with Nocilla
After this whole setup (which more or less you'll need to do only once)  we're ready to write first tests! 
I'll use Nocilla to test WebService class I created to handle JSON services. First test will check if completion handler is passed right data when server responds with a proper JSON.

```swift
func test_fetchParameters_200ProperJSON_CallsCompletionHandlerWithJSONObject() {
    // arrange
    let service = makeJSONWebService(baseURLString: baseURLString, defaultParameters: emptyParameters)
    stubRequest("GET", baseURLString).andReturn(200).withBody("{\"ok\":1}")
    // act
    service.fetch(emptyParameters) { result in
        self.expectation.fulfill()

        // assert
        XCTAssertEqual(["ok": 1], result.data()!)
    }
    waitForExpectationsWithTimeout(0.1, handler: nil)
}
```

Stubbing is straighforward. Start with `stubRequest` and define HTTP method and URL with either a string or a regex, then define request/response headers, payload, and response status code. In this example unit of work we're testing is web service library fetch call. Scenario/state under test is 200 OK response from server with JSON payload. As a result we're expecting completion handler to be called with JSON parsed to dictionary. Other scenarios to test could include handling of the 404 response:

```swift
stubRequest("GET", baseURLString).andReturn(404)
```

or testing malformed JSON data reponse:

```swift
stubRequest("GET", baseURLString).andReturn(200).withBody("{1}")
```

There are many more examples of stubbing requests with Nocilla's elegant DSL on [github](https://github.com/luisobo/Nocilla#stubbing-requests). Those include even responding with data recorded with curl and failing requests with specific `NSError`.

## Tests Structure and Naming
Notice the Arrange-Act-Assert pattern that helps to keep tests clean. In Arrange all needed objects/dependencies are set up, then method under test is invoked (Act) and finally we assert results/the state of the system under test (Assert). More details about the pattern [here](http://www.arrangeactassert.com/why-and-what-is-arrange-act-assert/). Another thing worth noting is the omition of the assertion message. I've used following standard for naming unit test ([more](http://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html) [details](http://osherove.com/blog/2012/5/15/test-naming-conventions-with-unit-of-work.html)):
> `UnitOfWork_StateUnderTest_ExpectedBehavior`


***

<div class="footnotes"><ol>
    <li class="footnote" id="fn:1">
        <p>Which would be impossible anyway as XCTestCase default initializer takes in NSInvocation, which is "unavailable".<a href="#fnref:1" title="return to article"> ↩</a><p>
    </li>
</ol></div>

