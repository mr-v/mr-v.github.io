---
layout: post
title: LLDB toolbox&#58; debugging views with private APIs in Swift
permalink: lldb-debugging-views-with-private-apis-in-swift
comments: true
---

Quick overview of some of the view debugging private APIs in the SDK and how to use them with Swift.

<!--more-->

- `+[UIViewController _printHierarchy]` - prints view controller hierarchy:

~~~objc
(lldb) po [UIViewController _printHierarchy]
<UITabBarController 0x7ffb02c98720>, state: appeared, view: <UILayoutContainerView 0x7ffb02d53140>
   | <UINavigationController 0x7ffb0301ac00>, state: appeared, view: <UILayoutContainerView 0x7ffb02cad460>
   |    | <ViewController 0x7ffb02c98af0>, state: appeared, view: <UIView 0x7ffb02ca9870>
   | <UINavigationController 0x7ffb03035a00>, state: disappeared, view:  (view not loaded)
   |    | <SettingsViewController 0x7ffb02d4ccc0>, state: disappeared, view:  (view not loaded)
~~~

- `-[UIView recursiveDescription]` - prints view hierarchy. Note that nowadays it may be more convenient to use Xcode's visual debugger for going over views structure.
- `-[UIView _autolayoutTrace]` - prints view hierarchy for the whole app with hints about ambiguous layout
- `-[UIView _recursiveAutolayoutTraceAtLevel:]` - prints hierarchy with hints only for the specified view. Just use `0` for paramater, it only determines level of indendation.

When breaking inside Swift code, we're relying on Swift compiler to parse expression. Using Objective-C syntax won't fly.

~~~objc
(lldb) po [UIViewController _printHierarchy]
error: <EXPR>:1:19: error: expected ',' separator
[UIViewController _printHierarchy]
                  ^
~~~

It's private, so Swift can't see it as well.

~~~swift
(lldb) po UIViewController._printHierarchy()
error: <EXPR>:1:1: error: type 'UIViewController' has no member '_printHierarchy'
UIViewController._printHierarchy()
^~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~
~~~

The method is there, but LLDB needs to be informed to use Clang to get to it. That can be achieved with `-l/--language` switch. `po` is in fact `expression -O --`. After adding language information proper way to call `_printHierarchy` becomes:

~~~objc
expression -l objc++ -O -- [UIViewController _printHierarchy]
~~~

Writing `expression -l objc++ -O --` each time we want to use SDK's private API is no fun <sup id="fnref:1"><a href="#fn:1" rel="footnote">1</a></sup>. To save keystrokes add an alias to `~/.lldbinit` file:

`command alias spo expr -l objc++ -O --`

To reload `lldbinit` without restarting Xcode execute : `command source ~/.lldbinit`.

Quick test inside Swift frame confirms that everything works.

~~~objc
(lldb) spo [UIViewController _printHierarchy]
<UITabBarController 0x7febdb4264a0>, state: appeared, view: <UILayoutContainerView 0x7febdb438230>
   | <UINavigationController 0x7febdc015800>, state: appeared, view: <UILayoutContainerView 0x7febdb43acb0>
   |    | <LLDBTesting.SwiftViewController 0x7febdb6073e0>, state: appeared, view: <UIView 0x7febdb5c6710>
   | <UINavigationController 0x7febdc014400>, state: disappeared, view:  (view not loaded)
   |    | <SettingsViewController 0x7febdb429a80>, state: disappeared, view:  (view not loaded)
~~~

Happy debugging!

<div class="footnotes"><ol>
    <li class="footnote" id="fn:1">
        <p>Writing "e -l objc++ -O --" isn't optimal as well.<a href="#fnref:2" title="return to article"> ↩</a><p>
    </li>    
</ol></div>
