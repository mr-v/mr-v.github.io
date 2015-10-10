---
layout: post
title: ARAnalytics&#58; analytics, clean code and better crash logs
permalink: aranalytics-analytics-clean-code-and-crash-logs
comments: true
---

_The original post is published on the Tivix blog at the following URL: http://www.tivix.com/blog/aranalytics-analytics-clean-code-crash-logs-on-ios/_

Tracking how users are using your app is a must. When looking for improvements we should rely on data, instead on just a hunch. There are a lot of analytics services and each of them has its own SDK. Integrating different analytics services,  adding code for logging events and screen views, is at best tedious and bloats code in places that are already susceptible to it (Hello, [Massive View Controllers](https://twitter.com/colin_campbell/status/293167951132098560)). Switching to another service leads to changing code all over the app. By default it's also impossible to just quickly try out a new provider, without repeating work that's already been done.
 
But it's not all doom and gloom! ARAnalytics is an Objective-C library helping with integrating those services into iOS and OS X apps. It's a wrapper, that reduces an overhead of learning yet another framework. Currently it supports 32 different services. Just integrate one of the providers available in ARAnalytics, set it up with API key and it's done. Library has all the standard  tracking features for events, timed events, screen views, and errors. Thanks to using one consistent API, developers can add and drop support for analytics services at will, without going through the hassle of making code changes in many places.

Below I'll explain how to set up ARAnalytics and take advantage of its great features that will help keep code DRY and get more insight from crash logs.

<!--more-->

## Integration using CocoaPods
Most straightforward way to integrate both Hockey App and Google Analytics would be:

1. Add references to the subspecs in Podfile

```ruby
pod 'ARAnalytics', :subspecs => ['HockeyApp', 'GoogleAnalytics']
```

2. Setup API keys

```objc
[ARAnalytics setupWithAnalytics: @{
      ARHockeyAppBetaID : @"KEY",
      ARGoogleAnalyticsID : @"KEY"
   }];
```

and it's good to go. This is nice to have, but the selling point of ARAnalytics is its DSL.

## Tracking events with DSL
DSL support helps with cleaning up the code even more and gives a lot of flexibility. User interactions can be tracked without cluttering view controller code. After adding `ARAnalytics/DSL` subspec, new setup method is available: `+[ARAnalytics setupWithAnalytics:configuration:]`. New parameter is a dictonary that takes in configuration for tracking events and screens. For the `ARAnalyticsTrackedEvents` key pass an array of tracked event details for each class. Let's setup a few events for `DetailViewController` and `MasterViewController` classes.

```objc
- (NSDictionary *)analyticsConfiguration {
    return @{ARAnalyticsTrackedEvents: @[
                     @{ARAnalyticsClass: DetailViewController.class,
                       ARAnalyticsDetails: @[@{ARAnalyticsEventName: @"show details", ARAnalyticsSelectorName: ARAnalyticsSelector(showDetails)}]},
                     @{ARAnalyticsClass: MasterViewController.class,
                       ARAnalyticsDetails: @[
                               @{ARAnalyticsEventName: @"edit", ARAnalyticsSelectorName: ARAnalyticsSelector(edit)},
                               @{ARAnalyticsEventName: @"save", ARAnalyticsSelectorName: ARAnalyticsSelector(save) }
                               ]}
                     ]
             };
}
```

Simply set event's name and refer to the selector that triggers logging. I admit that all the prefixes and different brackets can obscure what's actually going on here. Will do a little refactoring to clean this up:

```objc
- (NSDictionary *)analyticsConfiguration {
    return @{ARAnalyticsTrackedEvents: @[[self masterViewControllerTrackedEvents],
                                         [self detailViewControllerTrackedEvents]]};
}


- (NSDictionary *)masterViewControllerTrackedEvents {
    return @{ARAnalyticsClass: DetailViewController.class,
             ARAnalyticsDetails: @[
                     @{ARAnalyticsEventName: @"show details", ARAnalyticsSelectorName: ARAnalyticsSelector(showDetails)}]};
}

- (NSDictionary *)detailViewControllerTrackedEvents {
    return @{ARAnalyticsClass: MasterViewController.class,
             ARAnalyticsDetails: @[
                     @{ARAnalyticsEventName: @"edit", ARAnalyticsSelectorName: ARAnalyticsSelector(edit)},
                     @{ARAnalyticsEventName: @"save", ARAnalyticsSelectorName: ARAnalyticsSelector(save) }
                     ]};
}
```

Now it reads much better. `analyticsConfiguration` has high level overview of what is tracked. For details of specific class developer can go level below to methods encapsulating events for each class. This has additional benefit of being able to quickly answer whether something is tracked or not, without searching through the view controllers.

## Tracking screen views with DSL
Let's compare how ARAnalytics and Google Analytics SDK differ in aspect of tracking screen views. Google Analytics SDK requires developer to add this all over the app: `[[GAI sharedInstance] defaultTracker] set:kGAIScreenName value:@"Home Screen"];`. As we've already established sprinkling codebase with analytics calls can be avoided. Google provides way for automatic screen measurement within its SDK. Just change all the view controllers in the app to inherit from `GAITrackedViewController`. Wait a sec. That's quite a dependency! Not to mention - how do we track third party screen views? By hand? No, thanks!
Here's where ARAnalytics really starts to shine. All screens can be tracked with the following code.

```objc
- (NSDictionary *)analyticsConfiguration {
    return @{ARAnalyticsTrackedEvents: @[[self masterViewControllerTrackedEvents],
                                         [self detailViewControllerTrackedEvents]],
             ARAnalyticsTrackedScreens: [self trackedScreens]};
}

- (NSDictionary *)trackedScreens {
    return @{ARAnalyticsTrackedScreens: @[@{
                                              ARAnalyticsClass: UIViewController.class,
                                              ARAnalyticsDetails: @[@{ARAnalyticsPageNameKeyPath: @"title"}]
                                              }
                                          ]};
}
```

All screen views will be automatically logged now. No matter if it's our code, third party library or system framework. What's pretty cool is that developer doesn't need to remember about logging views when adding a new screen. It will just work. However `title` may not always be set and I want view controller's specific class to be logged.

```objc
@implementation UIViewController (ClassName_tvx)
- (NSString *)className_tvx {
    return NSStringFromClass([self class]);
}
@end

...

- (NSDictionary *)trackedScreens {
    return @{ARAnalyticsTrackedScreens: @[@{
                                              ARAnalyticsClass: UIViewController.class,
                                              ARAnalyticsDetails: @[@{ARAnalyticsPageNameKeyPath: ARAnalyticsSelector(className_tvx)}]
                                              }
                                          ]};
}
```
With help of a small category a class name now gets logged. To weed out `UINavigationController`, `UITabBarController` and alike, for `ARAnalyticsShouldFire` key add block that filters unwanted values.

```objc
- (NSDictionary *)trackedScreens {
    NSArray *ignoredClasses = @[[UINavigationController class], [UITabBarController class]];
    return @{ARAnalyticsClass: UIViewController.class,
             ARAnalyticsDetails: @[@{ARAnalyticsPageNameKeyPath: ARAnalyticsSelector(className_tvx),
                                     ARAnalyticsShouldFire: ^BOOL(UIViewController *controller, NSArray *parameters) {
                                         for(Class c in ignoredClasses) {
                                             if ([controller isKindOfClass:c]) {
                                                 return NO;
                                             }
                                         }
                                         return YES;
                                     }}]
             };
}
```

## Enhancing Hockey App crash logs
Now for the cherry on the top. ARAnalytics works especially well with HockeySDK. All the data that is sent to analytics services can be attached to crash logs! Having such breadcrumbs is a blessing, especially when stack trace doesn't say much and you can't reproduce the crash. With this help developers can retrace user's steps and have better chance of improving app's stability. I recommend including other information that's useful for the debugging: app's lifecycle events, going offline/online, and network calls. To add custom debug data use library's own logger `ARLog`:

```objc
- (void)logAppLifecycleEvent:(NSNotification *)notification {
    ARLog(@"lifecycle event: %@", notification);
}
```

Extra data is avaialbe in Description tab of Hockey crash logs. Note: not all logs will contain description due to implementation:
>  (...) `syslogd` will not keep logs around for a long time, as such you should only expect logs of people that re-start the application immediately after the application crashing.


 <br />
ARAnalytics is a great library that helps with keeping code clean, separating analytics from rest of the app, and tracking those evil crashes.
