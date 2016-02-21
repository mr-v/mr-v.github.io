---
layout: page
title: iOS Development Toolbox
comments: true
---

Some things I find useful during everyday work: it may be API design guideline, debugging tool, command-line utility or some cool Xcode plugin.

# Code

## Swift

### Style/syntax
Where Swift is going and how it relates to existing Objective-C codebases.

[API Design Guidelines](https://swift.org/documentation/api-design-guidelines.html)

[Better Translation of Objective-C APIs Into Swift](https://github.com/apple/swift-evolution/blob/master/proposals/0005-objective-c-name-translation.md)

Example of applying guidelines to existing code: 
[Apply API Guidelines to the Standard Library](https://github.com/apple/swift-evolution/blob/master/proposals/0006-apply-api-guidelines-to-the-standard-library.md)

### Writing Documentation

> Write a documentation comment for every declaration. Insights gained by writing documentation can have a profound impact on your design, so don’t put it off.

> If you are having trouble describing your API’s functionality in simple terms, you may have designed the wrong API.
bloc

[Swift Documentation](http://nshipster.com/swift-documentation/) - NSHipster article covering most useful documentation markup parts for day to day work.

[Thorough Markup Formatting Reference from Apple](https://developer.apple.com/library/mac/documentation/Xcode/Reference/xcode_markup_formatting_ref/)

### Other

[Profiling your Swift compilation times](http://irace.me/swift-profiling/)


## Libraries

[Creating your first iOS Framework](https://robots.thoughtbot.com/creating-your-first-ios-framework)

Checking out libraries:

- [cocoapods-try](https://github.com/CocoaPods/cocoapods-try) - "quickly try the demo project of a Pod" - `pod try POD_NAME`.

- [ThisCouldBeUsButYouPlaying](https://github.com/neonichu/ThisCouldBeUsButYouPlaying) - amazing combo of amazing things: Swift playgrounds and open source. Specify libraries you want to play around and get playground with them. Supports both CocoaPods `pod playgrounds RxSwift,RxCocoa` and Carthage `carthage-play Alamofire/Alamofire`.

### RxSwift

[RxSwift](https://github.com/ReactiveX/RxSwift) - lots of documentation and playground

[Functional Reactive Intuition - Swift edition (blog)](http://itchingpixels.com/blog/functional-reactive-intuition-swift/) - nice example of using a bunch of different operators, mixing and matching different inputs: timers, gesture recognizers and stuff.

[Functional Reactive Programming with RxSwift (video)](https://realm.io/news/slug-max-alexander-functional-reactive-rxswift/) - another great practical example, this one: chaining async calls.

[RxMarbles](http://rxmarbles.com) - Interactive diagrams of Rx Observables


### Other

Parsing JSON:

- Objective-C: [KZPropertyMapper](https://github.com/krzysztofzablocki/KZPropertyMapper)

- Swift: [Unbox](https://github.com/JohnSundell/Unbox) - simple, no weird operators, works as advertised.


[Then](https://github.com/devxoul/Then) - "✨ Super sweet syntactic sugar for Swift initializers." True! ✨

[Cartography](https://github.com/robb/Cartography) - "A declarative Auto Layout DSL for Swift" - combines best of both worlds: succint, easy to follow syntax and type safety. Goodbye, VFL!

[OAStackView](https://github.com/oarrabi/OAStackView) - UIStackView goodnes ported back to iOS 7/8.

[R.swift](https://github.com/mac-cain13/R.swift) - "Get strong typed, autocompleted resources like images, fonts and segues in Swift projects"

### Dependency Management

[Carthage](https://github.com/Carthage/Carthage):

- [video overview](https://realm.io/news/swift-dependency-management-with-carthage/)
- `carthage update --platform iOS --no-use-binaries`

[CocoaPods](https://cocoapods.org/)

[Swift Package Manager](https://swift.org/package-manager) (waiting to use it until Swift 3.0 arrives)


# Tools

# IDE

## [AppCode](https://www.jetbrains.com/objc/)
IDE with amazing Objective-C support: many refactorings, fixing _old and syntax with hitting ⌥-Enter, "Show Usages", "Show History for Selection."
Unfortunately at this time only so-so Swift support.

Tips:

> Use ⌥-Space (View | Quick Definition), to quickly review definition or content of the symbol at caret, without the need to open it in a new editor tab.

[Workshop](https://github.com/JetBrains/appcode-workshop) - learn your way around the AppCode, pick up some shortcuts along the way.

[List of nice things about AppCode.](https://github.com/orta/AppCode/blob/master/delight.md)


## Xcode


### Xcode Plugins

[Alcatraz](http://alcatraz.io/) - plugins package manager.

[Refactorator](https://github.com/johnno1962/Refactorator) - rename refactoring in Swift! Setting keyboard shortcut:

![](https://cloud.githubusercontent.com/assets/1786033/12243180/0a0839ac-b895-11e5-93f8-7caaceb22250.png)



[Quick Jump](https://github.com/wiruzx/QuickJump) - Jumping to any character/word/line you see on screen in an instant. To appreciate it's coolness check out this [video](https://www.youtube.com/watch?v=UZkpmegySnc) of original ace-jump for Emacs. 

[BlockJump](https://github.com/tyeen/BlockJump)

> This plug-in lets you jump between methods, or other items in the source editor.

```
^ + [ : jump up
^ + ] : jump down
```

### Other

[IconOverlaying](https://github.com/krzysztofzablocki/IconOverlaying) - "Build informations on top of your app icon."

# OS X

[Hyper Key](https://msol.io/blog/tech/work-more-efficiently-on-your-mac-for-developers/#the-hyper-key) - using it to simplify some of the shortcuts (e.g. re-run last unit tests in Xcode) and switch between the apps (e.g. hyper + X - opens Xcode, hyper + S - opens Safari).

[Quick Look for Markdown files](https://github.com/toland/qlmarkdown)

[Provisioning](https://github.com/chockenberry/Provisioning) - "A Quick Look plug-in for .mobileprovision files"

# Terminal

[autojump](https://github.com/wting/autojump) - "A cd command that learns - easily navigate directories from the command line". Using this I can jump quickly to any folder I've visited. Let's say I want to open folder with open source projects: running `j opens` takes me to `~/Developer/open_source` - neat!

[tree](http://mama.indstate.edu/users/ice/tree/) - "recursive directory listing command that produces a depth indented listing of files". On OS X can be installed with brew: `brew install tree`.


[selecta](https://github.com/garybernhardt/selecta) - "A fuzzy text selector for files and anything else you need to select. Use it from vim, from the command line, or anywhere you can run a shell command."

![](https://camo.githubusercontent.com/0a2501433a908cd8c8012a273df6efa26c432939/68747470733a2f2f7261772e6769746875622e636f6d2f676172796265726e68617264742f73656c656374612f6d61737465722f64656d6f2e676966)

I've created alias that combines `tree` and `selecta` to open any file from current directory and its subdirectories: `alias tsof='open "$(tree -if | selecta)"'`.

---

## [Charles](http://www.charlesproxy.com/)
Proxy for monitoring network traffic. Since Apple is pushing privacy [http://www.apple.com/customer-letter/](very hard) ([App Transport Security](http://useyourloaf.com/blog/app-transport-security/)) and [Let's Encrypt](https://letsencrypt.org/) makes it easy "to obtain a trusted certificate at zero cost.", encrypted traffic will be more and more prevalent, [How to inspect SSL traffic with Charles](http://nsscreencast.com/episodes/73-ssl-pinning) comes in handy.

## [Reveal](http://revealapp.com/)

Apple is trying to catch up with the original visual debugger, but so far Reveal experience is much smoother, allows multiple snapshots and has live editing.

## [xScope](http://xscopeapp.com/)

_Dimensions_ tool alone is amazing, especially when all designer shares with you is a flat png, see [video](https://www.youtube.com/watch?v=BwctGB7w0G4).

## [Hopper](http://hopperapp.com/)

Great for peeking under the hood of SDK: [introduction](http://www.bartcone.com/new-blog/2014/11/26/hopper-lldb-for-ios-developers-a-gentle-introduction).

> Hopper is a reverse engineering tool for OS X and Linux, that lets you disassemble, and decompile your 32/64bits Intel Mac, Linux, Windows and iOS executables!

> Even if Hopper can disassemble any kind of Intel executable, it does not forget its main platform. __Hopper is specialized in retrieving Objective-C information__ in the files you analyze, like selectors, strings and messages sent.


<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/modocache">@modocache</a> You can use the <b>⌥⇧X</b> shortcut: it searches all the references to the selector associated to the current method.</p>&mdash; bSr43 (@bSr43) <a href="https://twitter.com/bSr43/status/670662694459043841">November 28, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## [Macdown](http://macdown.uranusjr.com/)

Markdown editor. Previously used Brackets + extension for this, Macdown works much better.
